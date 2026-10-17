# Recursive Length Prefix (RLP) Encoding: How Ethereum Serializes Execution-Layer Data

> **Stage:** 03 — Storage and Data Layout  
> **Priority:** 2  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-17

## 1. Overview

**Recursive Length Prefix (RLP)** is a serialization format used extensively by Ethereum's execution layer. It encodes byte strings and recursively nested lists of items into a byte sequence. Prefix bytes tell a decoder whether an item is a single byte, a byte string, or a list, and how many bytes belong to that item.

RLP deliberately keeps the structural rules small. It does **not** automatically preserve high-level types such as Solidity's `uint256`, `address`, or `string`, and it does not include field names. A protocol or schema above RLP defines what each decoded byte string means. In Ethereum conventions, integers are represented as minimal-length, big-endian byte strings.

RLP matters to developers because execution-layer data—including Merkle-Patricia trie nodes and various protocol structures—is serialized with it. It is a client/protocol-level format, not the format Solidity automatically uses for every external function call.

![RLP prefixes identify a single byte, short or long byte strings, and short or long lists](../../assets/diagrams/rlp-encoding-overview.svg)

## 2. Why It Matters to a Developer

Understanding RLP helps when you:

- inspect raw Ethereum execution-layer data or debug client and RPC tooling;
- reason about the node encodings used in Merkle-Patricia tries and the data committed to by their roots;
- work with transaction or block serialization, while distinguishing legacy transaction encoding from typed transaction envelopes;
- validate RLP data in client libraries or in specialized on-chain proof-verification code.

A crucial boundary: **RLP is not Solidity ABI encoding**. A contract call normally uses ABI rules for the selector and arguments. The EVM does not automatically decode RLP when a contract receives calldata; any RLP parsing in a contract must be explicitly implemented or provided by a library.

## 3. The Data Model

RLP has two structural kinds of item:

- **Byte string:** an arbitrary sequence of bytes. It may contain UTF-8 text, integer bytes, an address, or any other data—but RLP itself does not know which.
- **List:** an ordered sequence of RLP items. Each child item is encoded recursively, so lists can contain other lists.

The encoding is deterministic when the same input structure is represented using the required canonical form. The list order is preserved. RLP does not define a general map/dictionary type; a higher-level protocol must define how key-value pairs and ordering are represented.

## 4. The Five Prefix Cases

The first byte of each item determines how to parse it. Lengths are measured in **bytes**, not characters.

| Prefix / case | Meaning | Encoded form |
| --- | --- | --- |
| `0x00–0x7f` | One byte with value below `0x80` | The byte itself; no extra prefix |
| `0x80–0xb7` | Byte string with length 0–55 bytes, except the single-byte shortcut above | `0x80 + length`, then payload |
| `0xb8–0xbf` | Byte string longer than 55 bytes | `0xb7 + length-of-length`, length bytes, then payload |
| `0xc0–0xf7` | List whose encoded payload totals 0–55 bytes | `0xc0 + payload length`, then concatenated encoded children |
| `0xf8–0xff` | List whose encoded payload totals more than 55 bytes | `0xf7 + length-of-length`, payload-length bytes, then concatenated encoded children |

For a long string or list, the length itself is encoded as the shortest big-endian byte sequence. The prefix records how many bytes are used to represent that length. RLP encodings are limited to payload lengths below `2^64` bytes.

### 4.1 Single-byte values

If the byte is in the range `0x00` through `0x7f`, it encodes as itself.

- The byte `0x64` is encoded as `64`.
- The byte `0x7f` is encoded as `7f`.
- A byte with value `0x80` does **not** use this shortcut: it becomes `81 80`, because the payload byte is outside the shortcut range.

### 4.2 Short byte strings

For a byte string of 0–55 bytes, prepend one byte equal to `0x80 + payload length`, unless the string is exactly one byte whose value is below `0x80`.

Examples:

- Empty byte string: `80`
- `"dog"`: `83 64 6f 67` — `83` means three payload bytes follow.

### 4.3 Long byte strings

For a payload longer than 55 bytes:

1. Encode the payload length in minimal big-endian form.
2. Let `L` be the number of bytes in that length representation.
3. Prefix the item with `0xb7 + L`.
4. Append the length bytes, then the payload.

For a 56-byte payload, the length is `0x38`, represented by one byte. The prefix is therefore `0xb8`, giving `b8 38` followed by the 56 payload bytes.

For a 1,024-byte payload, the length is `0x0400` and uses two bytes, so the prefix and length are `b9 04 00`, followed by the payload.

### 4.4 Lists

To encode a list, first RLP-encode each child, concatenate those encoded child items, and then prefix that combined payload using the list rules. The length of a list is the length of the **encoded children**, not the number of children.

- Empty list: `c0`.
- A list with eight bytes of encoded children starts with `c8`.
- If the combined encoded payload is longer than 55 bytes, use the long-list form beginning in the `0xf8–0xff` range.

## 5. Worked Examples

### Example A: Encode the byte string `"dog"`

The ASCII bytes for `dog` are:

```text
64 6f 67
```

There are three bytes, so the short-string prefix is `0x80 + 3 = 0x83`.

```text
Input payload:  64 6f 67
RLP output:     83 64 6f 67
```

### Example B: Encode the list `["cat", "dog"]`

Each word is encoded independently:

```text
"cat" -> 83 63 61 74
"dog" -> 83 64 6f 67
```

The encoded children total eight bytes. The list prefix is `0xc0 + 8 = 0xc8`.

```text
RLP output: c8 83 63 61 74 83 64 6f 67

The first byte, `c8`, is the list prefix: eight bytes of encoded children follow.
```

The decoder sees `c8`, reads exactly eight bytes of list payload, then decodes two child strings inside that boundary.

### Example C: Empty values and the integer-zero convention

```text
Empty byte string:  80
Empty list:         c0
Single byte 0x00:   00
Single byte 0x0f:   0f
```

This distinction is important. RLP's raw structural input is byte strings and lists; the convention used by Ethereum for integer values converts an integer to its shortest big-endian byte representation before RLP-encoding it. Under that convention, integer zero is represented by the empty byte string, producing `80`. It is **not** the same as a byte string containing one zero byte, which encodes as `00`.

For example, integer 1 encodes as `01`, integer 15 as `0f`, integer 128 as `81 80`, and integer 1,024 as `82 04 00`.

## 6. How Decoding Works

An RLP decoder reads the prefix and determines the item's kind and payload boundary:

1. If the prefix is `0x00–0x7f`, the prefix byte is the item itself.
2. If it indicates a short string or short list, the prefix gives the payload length directly.
3. If it indicates a long string or long list, the prefix gives the number of bytes that encode the payload length. The decoder reads that length, then identifies the exact payload boundary.
4. A list decoder recursively decodes child items only within the list's payload boundary.
5. The higher-level protocol interprets the resulting byte strings according to its expected schema.

RLP tells the decoder **how to separate structure and payload**. It does not, by itself, tell the decoder that a particular 20-byte payload is an Ethereum address or that a byte string represents a nonce.

## 7. Where RLP Fits in Ethereum

RLP is used extensively in the execution layer, including the encoding of nodes in Ethereum's Merkle-Patricia tries. The exact interpretation of an RLP item is defined by the higher-level structure that contains it.

Transactions require one important distinction:

- **Legacy transactions** are represented as RLP-encoded lists of transaction fields.
- **Typed transactions** use the EIP-2718 envelope form: a transaction-type byte followed by a type-specific payload. Many current typed transaction payloads use RLP, but the envelope standard does not require every possible type to use the same payload encoding.

RLP is also distinct from **SSZ (Simple Serialize)**, which is widely used in Ethereum's consensus layer and is designed to support Merkleization. Do not assume that every Ethereum data structure uses RLP.

## 8. RLP vs ABI vs SSZ

| Format | Main role | What it defines |
| --- | --- | --- |
| RLP | Ethereum execution-layer serialization | Nested byte strings and lists with length prefixes |
| Solidity ABI | Contract function calls, returns, and event data | Type-aware encoding rules for Solidity-level values |
| SSZ | Ethereum consensus-layer serialization | Schema-based serialization with Merkleization-friendly rules |

These formats solve different problems. Encoding a Solidity value with `abi.encode` does not produce RLP in general, and RLP encoding does not automatically create valid ABI calldata.

## 9. Common Mistakes and Security Considerations

- **Assuming RLP carries type information.** A byte string is just bytes until a higher-level schema interprets it.
- **Confusing empty bytes, a zero byte, integer zero, and an empty list.** Under Ethereum's integer convention, integer zero encodes as `80`; raw byte `00` encodes as `00`; an empty list encodes as `c0`.
- **Using non-canonical encodings.** Long-form encodings should not be used when the short form applies. Integer byte sequences must not contain unnecessary leading zero bytes. Enforce the canonical rules expected by the protocol.
- **Trusting declared lengths.** Reject truncated payloads, impossible lengths, arithmetic overflow, and items that extend beyond their enclosing list. Avoid allocations based on unchecked lengths.
- **Ignoring trailing bytes.** When a protocol expects exactly one top-level item, ensure the decoder consumed exactly the expected input.
- **Treating successful decoding as proof of authenticity.** RLP is serialization, not a signature, hash, or Merkle proof. Integrity and authenticity require separate cryptographic validation.
- **Assuming every transaction payload is plain RLP.** Typed transaction envelopes must be parsed according to their transaction type and relevant specification.
- **Using an ad hoc parser in a smart contract.** If an on-chain proof verifier needs RLP parsing, review the parser carefully, bound resource use, and test malformed as well as valid inputs.

## 10. Key Takeaways

- RLP encodes byte strings and nested lists; a prefix identifies an item's kind and length.
- Payloads of up to 55 bytes use short forms; longer payloads include a length-of-length field.
- List length counts the concatenated **encoded** children.
- Ethereum integer values use minimal big-endian bytes before RLP encoding; under this convention, integer zero is the empty byte string.
- RLP is important for execution-layer data and trie nodes, but it is not the Solidity ABI and is not Ethereum's consensus-layer SSZ.
- Canonical decoding, strict boundary checks, and correct higher-level interpretation are essential.

## References

- [Ethereum.org — Recursive-length prefix (RLP) serialization](https://ethereum.org/developers/docs/data-structures-and-encoding/rlp/) — encoding cases, examples, integer conventions, and decoding overview.
- [Ethereum Yellow Paper, Appendix B](https://ethereum.github.io/yellowpaper/paper.pdf) — formal RLP definition.
- [Ethereum Execution Layer Specifications — RLP implementation](https://github.com/ethereum/ethereum-rlp) — RLP as used by the execution-layer specification, with implementation and tests.
- [go-ethereum RLP decoder](https://github.com/ethereum/go-ethereum/blob/master/rlp/decode.go) — implementation details and canonical-length checks.
- [EIP-2718 — Typed Transaction Envelope](https://eips.ethereum.org/EIPS/eip-2718) — distinction between the typed transaction envelope and the payload encoding.
