# Transaction Types (EIP-2718): Typed Transaction Envelopes

> **Stage:** 05 — Advanced Transactions and Gas  
> **Priority:** 2  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-17

## 1. Overview

Ethereum supports multiple transaction formats. The original **legacy transaction** format predates typed transactions and encodes fields such as nonce, gas price, gas limit, destination, value, input data, and signature fields using Recursive Length Prefix (RLP) encoding.

As Ethereum evolved, new features needed different fields and validation rules. Forcing every new feature into the original format would make backward compatibility harder. **[EIP-2718 — Typed Transaction Envelope](https://eips.ethereum.org/EIPS/eip-2718)** introduced a common envelope that lets each transaction type define its own payload format while preserving the legacy format.

For a typed transaction, the general wire format is:

```text
TransactionType || TransactionPayload
```

Here, `||` means byte concatenation. The first byte identifies the transaction type; the remaining bytes are interpreted according to that type's specification. The envelope is a shared framework, not one single new transaction with a fixed set of fields.

![Diagram comparing the legacy RLP transaction format with typed transaction envelopes, where a type byte selects the payload rules for transaction types 1 through 4](../../assets/diagrams/transaction-types-eip-2718.svg)

The transaction type matters when building wallets, clients, signing systems, block explorers, indexers, and smart-contract infrastructure. Software must understand the correct field layout, fee model, signing data, and validation rules for the type being handled.

## 2. The Legacy Transaction Format

A classic legacy transaction is RLP-encoded as a list similar to:

```text
RLP([
  nonce,
  gasPrice,
  gasLimit,
  to,
  value,
  data,
  v,
  r,
  s
])
```

The legacy raw transaction does **not** begin with a typed-envelope prefix such as `0x01` or `0x02`. Its first byte comes from the RLP encoding of the list.

Legacy transactions use a single `gasPrice` field rather than the separate EIP-1559-style `maxFeePerGas` and `maxPriorityFeePerGas` fields. Chain-aware signing can be represented by the legacy `v` value under EIP-155 rules; this signing protection did not require converting the on-wire legacy transaction into a typed envelope.

JSON-RPC APIs may expose a `type` property with a value such as `0x0` for legacy transactions. That API representation does not mean the raw legacy bytes contain a literal `0x00` type prefix.

## 3. The Typed Transaction Envelope

EIP-2718 separates two responsibilities:

- **Envelope:** identifies which transaction format the client should decode and validate.
- **Payload:** contains the fields and signature data defined by the selected transaction type.

For typed transactions, the payload is an opaque byte array from the perspective of EIP-2718 itself. A type-specific EIP defines how that payload is encoded and what every field means. Common Ethereum transaction types use RLP-encoded payload lists, but EIP-2718 does not require every future type to use RLP.

The type byte also helps maintain clear signing domains. EIP-2718 strongly recommends that a typed transaction's type byte be included at the start of the signed data, helping prevent a signature for one transaction format from being reused as a signature for another. Exact signing rules are defined by each transaction type's specification.

In the type-byte scheme described by EIP-2718, typed transaction type identifiers are in the low range up to `0x7f`. Legacy RLP transactions are distinguishable because their first byte falls in a different range. Type identifiers must not be interpreted in isolation: a client needs to know which formats are supported by the chain and implementation it is interacting with.

## 4. Important Ethereum Transaction Types

The following table summarizes several established Ethereum transaction formats. These identifiers are meaningful for the transaction type; they are not versions of Solidity contracts.

| Type | Name / standard | Main addition or defining feature |
|---:|---|---|
| Legacy (often API `0x0`) | Original legacy format | Original RLP transaction fields, including `gasPrice` |
| `0x01` | [EIP-2930](https://eips.ethereum.org/EIPS/eip-2930) — Access List Transaction | Adds an access list of addresses and storage keys, with associated gas accounting |
| `0x02` | [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559) — Fee-Market Transaction | Adds `maxPriorityFeePerGas` and `maxFeePerGas` to support the dynamic base-fee market |
| `0x03` | [EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) — Blob Transaction | Carries blob-related commitments and blob fee fields for data-availability use cases |
| `0x04` | [EIP-7702](https://eips.ethereum.org/EIPS/eip-7702) — Set Code Transaction | Adds an authorization list that lets an EOA authorize a delegation indicator to code at another address |

This is a focused list of major Ethereum transaction types, not a promise that every proposed type identifier is active on every EVM chain. Support depends on the network's protocol rules, fork activation, and client implementation.

### Type 1 — Access List (EIP-2930)

An access-list transaction includes an access list containing addresses and storage keys that the transaction expects to access. The list does not forbid accessing other addresses or slots. Rather, the format lets access-list entries be declared and accounted for under the gas rules introduced alongside EIP-2929.

Type 1 transactions use a `gasPrice` field; they do not use the EIP-1559 pair of max-fee fields. An access list may also appear in later typed formats, so the presence of an access list alone does not tell you that a transaction is type 1.

### Type 2 — EIP-1559 Fee-Market Transaction

A type-2 transaction uses two fee caps:

- **`maxFeePerGas`:** the maximum total per-gas price the sender is willing to pay, including the base fee.
- **`maxPriorityFeePerGas`:** the maximum per-gas priority fee offered to the block producer.

The effective price depends on the block's base fee and the transaction's fee caps. The sender does not necessarily pay the full maximum fee cap. The type-2 envelope still contains familiar fields such as chain ID, nonce, gas limit, destination, value, data, and access list, but its fee rules differ from legacy transactions.

### Type 3 — Blob Transaction (EIP-4844)

A blob transaction adds fields for a maximum blob gas fee and blob versioned hashes. The transaction is designed for data-availability payloads used by rollups.

Blob data is not ordinary calldata that EVM contract execution can read byte-for-byte. The execution layer can access the relevant commitments, while the blob data itself is handled through the blob data-availability mechanism. A type-3 transaction also has constraints such as requiring a non-null destination, so it is not a contract-creation transaction in the normal sense.

Blob transactions use both the ordinary execution-gas fee fields and a distinct blob-fee mechanism. Do not conflate blob gas with the EVM execution gas limit or assume both fee components behave identically.

### Type 4 — Set Code Transaction (EIP-7702)

A type-4 transaction includes an **authorization list** in addition to the usual transaction fields. Valid authorizations can set a delegation indicator in an EOA's code field, so operations involving that account can execute code delegated to an already-deployed address.

The outer transaction's sender nonce is distinct from the nonce included in each EIP-7702 authorization tuple. They are validated in different parts of the protocol and are not interchangeable.

EIP-7702 does not turn an EOA into a conventional deployed contract by copying the implementation's runtime code into the EOA. Instead, it installs a delegation indicator that points to code at another address. The delegated code is security-sensitive because it may execute operations with the EOA's authority under the EIP's rules.

## 5. Transaction Types and Receipts

EIP-2718 also defines how typed receipts are distinguished. A typed receipt uses the form:

```text
TransactionType || ReceiptPayload
```

The receipt payload is defined by the corresponding transaction type's rules. For existing typed transaction formats, the receipt payload commonly contains information equivalent to status, cumulative gas used, logs bloom, and logs, encoded according to the applicable specification.

The receipt type must match the type of the transaction it corresponds to. Legacy receipts retain their legacy representation rather than being wrapped in a modern typed prefix.

This distinction matters to clients and indexers that decode raw block data. A transaction's field layout and its receipt's layout are related, but they are not the same payload. Transaction types also affect how clients interpret fields before execution, while receipt data reports what happened after inclusion and execution.

## 6. How Nodes Identify the Format

At a high level, a client inspects the encoded transaction to decide whether it is a legacy transaction or a typed transaction:

1. **Legacy transaction:** the raw bytes are interpreted using the legacy RLP transaction format.
2. **Typed transaction:** the leading type byte identifies which transaction-specific decoder and validation rules apply.
3. **Payload decoding:** the client decodes the payload according to that type's specification.
4. **Signature and validity checks:** the client verifies the appropriate signature, chain/domain rules, fee constraints, nonce, balance, and type-specific conditions.
5. **Execution and receipt:** if included, the transaction follows the protocol's state-transition rules; the corresponding receipt uses the appropriate legacy or typed receipt representation.

This is a conceptual outline, not a complete consensus-validation procedure. Different transaction types can introduce different fields and checks, and a transaction can still be rejected or fail execution after format decoding.

## 7. Why This Design Is Useful

The typed-envelope model allows Ethereum to introduce new transaction capabilities without redesigning the original legacy transaction encoding for every change.

For a developer, this means:

- A wallet can display and sign different transaction types while using a shared envelope model.
- A client can dispatch to type-specific decoding and validation logic.
- A new EIP can add fields that the previous transaction format did not have.
- Existing legacy transactions can remain supported without pretending they have the same wire format as typed transactions.
- Indexers and libraries can preserve the type information needed to decode transactions and receipts correctly.

The trade-off is added implementation complexity. Infrastructure must support each type it claims to support, ensure type-specific fields are signed and validated correctly, and avoid silently treating all transactions as if they had the same field layout or fee model.

## 8. Current and Proposed Types: Don't Assume Every Type Is Active

EIP-2718 provides a framework for assigning transaction format identifiers, but does not itself mean every possible type value is implemented. New proposals can define candidate types before they become active network rules.

For example, **EIP-8202** proposes a scheme-agile transaction using type `0x05), but it is a draft proposal as of this article's review date. Do not treat that proposed identifier as a universally active Ethereum transaction format. Check the proposal's current status and the target chain's fork/client support before using a new type in production.

This is why clients should not infer support merely from a type number being in the allowed envelope range. The specification, network activation, and implementation support all matter.

## 9. Common Mistakes and Security Considerations

1. **Treating Legacy as `0x00 || payload`.** The original legacy transaction is RLP-encoded directly; an API's `type: 0x0` is a representation label, not a literal type prefix in the raw transaction.
2. **Assuming EIP-2718 fixes one payload format for every type.** It defines the envelope and leaves payload interpretation to each type specification.
3. **Assuming every typed payload must be RLP.** Current major types use RLP payload lists, but EIP-2718 keeps the payload opaque so future types can define other encodings.
4. **Ignoring the transaction type during signing.** Typed signatures should be separated by type-specific signing rules; omitting the type from a new format's signed data can create signature-reuse risks.
5. **Assuming the same fee fields exist in every type.** Legacy and type-1 use `gasPrice`, while type-2 and related later types use EIP-1559-style fee fields.
6. **Confusing execution gas with blob gas.** Type-3 blob transactions have a separate blob-fee mechanism.
7. **Treating blob data as ordinary calldata.** EVM execution does not have direct byte-for-byte access to the blob payload.
8. **Confusing type-4 authorization nonce and outer transaction nonce.** EIP-7702 includes nonce semantics in both places, but they serve separate purposes.
9. **Assuming every chain supports all types.** Type support depends on network rules, fork status, and client implementation.
10. **Decoding receipts as if they are transactions.** Receipt payload fields have different meanings and must follow the relevant receipt format.
11. **Using a draft EIP as if it were live.** Check status and implementation support before building around proposed transaction identifiers.
12. **Assuming valid encoding guarantees execution success.** A well-formed, correctly signed transaction can still fail other validity checks, remain pending, or revert during execution.

## 10. Developer Review Checklist

- [ ] Distinguish a legacy RLP transaction from the EIP-2718 typed envelope.
- [ ] Explain `TransactionType || TransactionPayload` and why the payload is type-specific.
- [ ] Identify types 1, 2, 3, and 4 and the EIPs that define them.
- [ ] Compare the fee fields used by legacy/type-1 and type-2/type-3/type-4 transactions.
- [ ] Explain the role of access lists, blob fields, and the EIP-7702 authorization list.
- [ ] Distinguish a transaction's sender nonce from an EIP-7702 authorization nonce.
- [ ] Explain why the typed receipt's type must match its transaction type.
- [ ] Check the transaction type's signing domain, network activation, and client support before integrating it.
- [ ] Distinguish a proposal's draft status from an active protocol rule.

## 11. Key Takeaways

- EIP-2718 introduces the typed transaction envelope: `TransactionType || TransactionPayload`.
- The envelope allows transaction types to define their own payload fields and validation rules while retaining the original legacy encoding.
- Legacy transactions are still encoded as legacy RLP lists; a JSON-RPC `type: 0x0` label does not add a zero prefix to the raw bytes.
- Type 1 (EIP-2930), type 2 (EIP-1559), type 3 (EIP-4844), and type 4 (EIP-7702) add different capabilities and fields.
- Typed receipts also carry the matching type identifier followed by type-specific receipt payload.
- Correct software must preserve type information and apply the right signing, fee, decoding, and validity rules.
- A type identifier in a proposal is not by itself evidence that the format is active on Ethereum mainnet or another target chain.

## References

1. **[EIP-2718 — Typed Transaction Envelope](https://eips.ethereum.org/EIPS/eip-2718)** — normative envelope, payload model, receipt encoding, backwards compatibility, and signing-domain guidance.
2. **[Ethereum.org — Transactions](https://ethereum.org/developers/docs/transactions/)** — overview of legacy and typed transaction formats.
3. **[EIP-2930 — Optional access lists](https://eips.ethereum.org/EIPS/eip-2930)** — type-1 fields, access-list structure, and gas accounting.
4. **[EIP-1559 — Fee market change for ETH 1.0 chain](https://eips.ethereum.org/EIPS/eip-1559)** — type-2 envelope, fee fields, and base-fee mechanism.
5. **[EIP-4844 — Shard Blob Transactions](https://eips.ethereum.org/EIPS/eip-4844)** — type-3 transaction fields, blob commitments, and blob gas fee model.
6. **[EIP-7702 — Set Code for EOAs](https://eips.ethereum.org/EIPS/eip-7702)** — type-4 transaction and authorization-list semantics.
7. **[EIP-8202 — Scheme-Agile Transactions](https://eips.ethereum.org/EIPS/eip-8202)** — draft example of a proposed additional typed transaction format.
8. **[Ethereum Execution APIs](https://github.com/ethereum/execution-apis)** — JSON-RPC transaction object representations for common Ethereum transaction types.
9. **[Transaction Lifecycle](01-transaction-lifecycle.md)** — the transaction's path from signing through inclusion, execution, and receipt.
10. **[Transaction Nonce](02-transaction-nonce.md)** — the sender sequence number that appears in multiple transaction types.
