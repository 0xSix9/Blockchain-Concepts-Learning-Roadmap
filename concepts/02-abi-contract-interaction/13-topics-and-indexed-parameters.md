# Topics and Indexed Parameters in Ethereum Events

> **Stage:** 02 — ABI and Contract Interaction  
> **Priority:** 1  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-17

## 1. Overview

Ethereum event logs contain **topics** and **data**. In Solidity, the <code>indexed</code> keyword determines which event parameters are stored in topics rather than in the log's data field. Topics make it possible for a node or application to search logs for particular event types and known indexed values.

For a standard non-anonymous event:

- <code>topics[0]</code> contains the Keccak-256 hash of the event's canonical signature.
- Up to three indexed parameters can occupy <code>topics[1]</code> through <code>topics[3]</code>.
- Non-indexed parameters are ABI-encoded into the log's <code>data</code> field.

A standard log can therefore have up to four topics total. An <code>anonymous</code> event does not automatically include the signature hash, so it can use up to four indexed parameters.

![Diagram showing a standard event log with topic 0 for the event signature hash, indexed arguments in subsequent topics, and non-indexed arguments ABI-encoded in the data field, followed by topic filtering with eth_getLogs](https://raw.githubusercontent.com/0xSix9/Blockchain-Concepts-Learning-Roadmap/main/assets/diagrams/topics-and-indexed-parameters.svg?sanitize=true)

[Open the Topics and Indexed Parameters diagram directly](https://github.com/0xSix9/Blockchain-Concepts-Learning-Roadmap/blob/main/assets/diagrams/topics-and-indexed-parameters.svg)

This design introduces an important trade-off: indexing a value makes it easier to filter logs by that value, but certain dynamic or complex values are stored as hashes and cannot be recovered from the topic alone.

For the broader event lifecycle, see [Events and Event Logs](12-events-and-event-logs.md). This article focuses on the exact topic layout, indexed argument encoding, and filter logic.

## 2. Topics in a Standard Event Log

Consider this event:

    event Transfer(
        address indexed from,
        address indexed to,
        uint256 amount
    );

When a contract emits it, a standard log has this conceptual layout:

    address = the contract that emitted the log

    topics[0] = keccak256("Transfer(address,address,uint256)")
    topics[1] = encoded from address
    topics[2] = encoded to address

    data = ABI-encoded amount

The <code>indexed</code> keyword is not included in the canonical event signature. Neither are parameter names. The event signature is built from the event name and canonical parameter types, in declaration order. For example, Solidity's canonical type for <code>uint</code> is <code>uint256</code>, so a declaration using <code>uint</code> and one using <code>uint256</code> use the canonical type <code>uint256</code> in the event signature.

The full 32-byte topic 0 hash is different from a function selector. A function selector normally uses only the first four bytes of a function-signature hash; a standard event uses the full 32-byte event-signature hash as its first topic.

### Why the emitting address matters

Topic 0 identifies an event signature, not a unique contract or a trusted source. Any contract can define an event with the same signature and emit a log matching that topic. When validating protocol activity, filter or verify the emitting contract address as well as the event topics and chain context.

## 3. How Indexed Parameters Are Encoded

The exact topic representation depends on the type.

### Value types that fit into a 32-byte topic

For value types such as addresses, integers, booleans, and fixed-size byte values, the indexed value is encoded into a 32-byte topic word according to the event ABI rules. Addresses and smaller unsigned integers are padded to the appropriate 32-byte representation; signed integers use sign extension as required by their ABI encoding.

Example:

    event OrderFilled(
        address indexed maker,
        uint256 indexed orderId,
        uint256 amount
    );

For this event, <code>maker</code> and <code>orderId</code> are represented directly in topics (as 32-byte encoded values), while <code>amount</code> is part of the data field. A consumer can decode the first two indexed values when it knows the ABI.

### Dynamic and complex types: the topic contains a hash

For indexed parameters that are dynamic or complex—including <code>string</code>, <code>bytes</code>, arrays, and structs—the topic contains a Keccak-256 hash of a special in-place encoding of the value, rather than the complete value itself.

    event DocumentRegistered(
        address indexed owner,
        string indexed documentName,
        string description
    );

Here, the indexed <code>documentName</code> topic stores a hash of the special encoding of the string. It does **not** store a normal ABI-encoded string that can be decoded back into the original text. The non-indexed <code>description</code> is ABI-encoded into <code>data</code> and can be decoded using the ABI.

This makes a dynamic value useful for filtering when the expected value is already known, but it prevents an application from reconstructing an arbitrary original value from the topic hash alone.

### The special encoding is not ordinary ABI encoding

When reasoning about hashes for indexed dynamic values, do not assume that hashing a normal ABI-encoded value always gives the topic value. The event ABI specification defines a special in-place encoding for indexed complex values.

For example, the encoding used before hashing a <code>string</code> or <code>bytes</code> value is its contents without a normal dynamic-value length prefix or padding. Arrays and structs follow their own specified concatenation and padding rules. When computing a topic off-chain, use a library implementation that follows the **indexed event parameter** encoding rules rather than guessing from standard function argument encoding.

Nested structures with multiple dynamic members can introduce encoding ambiguity in this special representation. Do not treat an indexed hash as a universally unambiguous commitment to an arbitrary nested data structure without considering the exact Solidity ABI encoding rules.

## 4. The Trade-Off: Indexed or Non-Indexed?

Choose indexed parameters based on how applications need to search for the event.

| Goal | Indexed parameter | Non-indexed parameter |
|---|---|---|
| Filter logs by a known address or identifier | Convenient: encoded value can be queried in topics | Not directly filterable through topics |
| Read arbitrary event values after fetching a log | Static value can be decoded from the topic; dynamic values are hashed | Value is ABI-encoded in data and can be decoded with the ABI |
| Use limited log topic space | Consumes a topic slot | Does not consume a topic slot |
| Keep the original dynamic string/bytes in the log's decoded data | Topic alone cannot recover it | Original value is decodable from data |

A reasonable design for a common query field is to mark it indexed—for example, an account address, token ID, or order ID. A description or a large string that consumers generally need to read is often better as a non-indexed parameter.

### Duplicating a dynamic value, when necessary

Sometimes an application needs both efficient filtering by a known dynamic value and the original value available in the log. It may define two event parameters for that purpose: one indexed parameter used for filtering, and one non-indexed parameter that carries the readable value.

For example:

    event NamePublished(
        bytes32 indexed nameHash,
        string name
    );

The contract must ensure both arguments represent the intended same value. This pattern uses additional event data and should be chosen deliberately; it is not necessary when consumers can reconstruct the value from other trusted sources.

## 5. Topic Filters with eth_getLogs

Ethereum JSON-RPC's <code>eth_getLogs</code> method accepts a filter object. The common filter fields include:

- <code>address</code>: one contract address or a list of emitting contract addresses.
- <code>fromBlock</code> and <code>toBlock</code>: the block range to search.
- <code>topics</code>: an ordered array of topic filters.

A simplified filter looks like this:

    {
      "address": "0xYourContractAddress",
      "fromBlock": "0x100",
      "toBlock": "0x200",
      "topics": [
        "0xEventSignatureHash",
        "0xEncodedIndexedValue"
      ]
    }

This example asks for logs from the selected address and block range whose topic 0 matches the event signature hash and whose topic 1 matches a particular indexed value. The example strings are placeholders, not real event hashes or addresses.

### Topic positions matter

Topics are positional. A value in filter position 1 filters <code>topics[1]</code>, not any topic containing that value.

For example, if the event is <code>Transfer(address indexed from, address indexed to, uint256 amount)</code>:

    "topics": [
      "0xTransferSignatureHash",
      "0xEncodedFromAddress",
      "0xEncodedToAddress"
    ]

This requests logs whose signature topic, sender topic, and recipient topic all match the supplied values.

### null means “wildcard at this position”

A <code>null</code> entry allows any value at that topic position while preserving the position of later filters.

To query transfers **to** a particular address, regardless of sender:

    "topics": [
      "0xTransferSignatureHash",
      null,
      "0xEncodedToAddress"
    ]

The <code>null</code> wildcard is important. Leaving out the second array element would shift the recipient filter into position 1 and filter the sender instead.

### OR alternatives within one position

A topic filter position can contain an array of alternatives, which means “match any of these values at this position.” For example, a query can match transfers where either the sender is one of two specified addresses:

    "topics": [
      "0xTransferSignatureHash",
      [
        "0xEncodedSenderA",
        "0xEncodedSenderB"
      ]
    ]

This is an OR within one topic position. Different topic positions are combined as AND conditions: the log must match the constraints for each specified position, while each alternatives array allows any of its values.

Some JSON-RPC providers impose limits on block ranges, response sizes, or filter complexity. Large backfills may need to be split into smaller ranges and retried carefully.

## 6. Querying Dynamic Indexed Values

Dynamic indexed values are searchable, but their topics contain hashes rather than the original values. To query one, an application must compute the correct topic hash according to the Solidity ABI's special indexed-event encoding rules.

For a simple event:

    event UserLabelChanged(
        address indexed user,
        string indexed label
    );

The log contains an encoded user address topic and a hash representing the indexed label. A filter can match a known expected label only if the application computes the same hash representation Solidity uses.

Do not automatically compute the value as <code>keccak256(abi.encode(label))</code> or <code>keccak256(abi.encodePacked(label))</code> without checking the ABI event-parameter encoding rules. The special encoding for indexed event parameters can differ from both general-purpose encoding methods, especially for compound values.

For dynamic values, the hash allows equality checks against a known candidate; it cannot provide the original value for unknown candidates. If an indexer must display arbitrary labels, store the label in a non-indexed parameter too or retrieve it from another trusted source.

## 7. Indexed Parameters in Anonymous Events

An event declared <code>anonymous</code> does not automatically place its signature hash into <code>topics[0]</code>. This changes both the available number of indexed parameter slots and how a consumer identifies the event.

- Standard non-anonymous event: up to three indexed parameters in addition to the signature hash, for up to four topics total.
- Anonymous event: up to four indexed parameters and no automatically generated signature topic.

A consumer cannot rely on the standard topic-0 signature filter for an anonymous event. It needs another way to distinguish the log, usually including the emitting contract address and an expected log layout. Anonymous events should be used only when the integration is intentionally designed for those rules.

## 8. Practical Solidity Example

The contract below emits a small event suited to queries by account and product ID:

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.24;

    contract PurchaseLog {
        event Purchased(
            address indexed buyer,
            uint256 indexed productId,
            uint256 quantity,
            string note
        );

        function recordPurchase(
            uint256 productId,
            uint256 quantity,
            string calldata note
        ) external {
            require(quantity > 0, "Zero quantity");

            emit Purchased(
                msg.sender,
                productId,
                quantity,
                note
            );
        }
    }

For a standard log of <code>Purchased</code>:

- <code>topics[0]</code> is the signature hash for <code>Purchased(address,uint256,uint256,string)</code>.
- <code>topics[1]</code> contains the buyer address in topic-word form.
- <code>topics[2]</code> contains the product ID in topic-word form.
- <code>data</code> contains the ABI-encoded quantity and note.

The note is non-indexed, so consumers can decode the full string from the data field. The contract's event does not update purchase or payment state by itself; it is a minimal example for demonstrating the emitted log layout.

An application interested in all purchases of product ID 42 from this contract could set the contract address, a block range, the event signature in topic 0, and the ABI-encoded integer 42 in topic 2, using <code>null</code> in topic 1 to match any buyer.

## 9. Common Mistakes and Security Considerations

1. **Treating topics as a bag of values.** Topic slots have fixed positions; a filter for topic 1 does not search all topics.
2. **Forgetting the event signature topic.** A standard non-anonymous event normally uses topic 0 for its signature, leaving up to three indexed arguments.
3. **Using the wrong canonical signature.** Parameter names and <code>indexed</code> markers are excluded; canonical parameter types and their order matter.
4. **Assuming indexed strings are readable.** Indexed dynamic values are hashed and usually cannot be recovered from the topic alone.
5. **Hashing the wrong encoding.** Special indexed event parameter encoding is not always the same as <code>abi.encode</code> or <code>abi.encodePacked</code>.
6. **Shifting topic positions after a wildcard.** Use <code>null</code> where a topic position should match any value but later positions still matter.
7. **Confusing OR and AND.** An array of alternatives within one topic position is OR; constraints at different positions are combined as AND.
8. **Assuming a signature topic proves the emitting contract is trusted.** Verify the address and chain context along with topics.
9. **Indexing every parameter by default.** Topics are limited; choose fields based on expected query patterns and readability needs.
10. **Ignoring provider query limits or chain reorganizations.** Backfill in manageable block ranges, deduplicate logs, and reconcile events if their blocks cease to be canonical.

## 10. Key Takeaways

- Topics are ordered 32-byte values used for event identification and indexed-argument filtering.
- For standard events, <code>topics[0]</code> is the full Keccak-256 hash of the canonical event signature.
- Up to three parameters can be indexed in a standard event; anonymous events can index up to four parameters.
- Static/value-type indexed arguments are encoded into topics; dynamic and complex indexed arguments are represented by a Keccak-256 hash of a special encoding.
- Non-indexed parameters are ABI-encoded in the log's data field and can be decoded with the correct ABI.
- Topic filters are positional: different positions combine as AND, arrays of alternatives within a position form OR, and <code>null</code> provides a wildcard.
- Indexed dynamic values are useful for filtering known values, but a hash does not reveal an arbitrary original string or byte array.

## References

- [Solidity Documentation — ABI Specification: Events](https://docs.solidity.org/en/latest/abi-spec.html#events)
- [Solidity Documentation — Encoding of Indexed Event Parameters](https://docs.solidity.org/en/latest/abi-spec.html#encoding-of-indexed-event-parameters)
- [Solidity Documentation — Contracts: Events](https://docs.solidity.org/en/latest/contracts.html#events)
- [Ethereum JSON-RPC — eth_getLogs and topic filters](https://ethereum.org/en/developers/docs/apis/json-rpc/#eth_getlogs)
- [Events and Event Logs](12-events-and-event-logs.md)

## Review Checklist

- [ ] Explain the standard event topic layout and the event-signature hash.
- [ ] Distinguish direct topic encoding for value types from hashed encoding for dynamic or complex types.
- [ ] Explain topic-position matching, wildcards, OR alternatives, and AND across positions.
- [ ] Build or interpret an `eth_getLogs` filter for a Solidity event.
- [ ] Explain anonymous events, indexed-value encoding pitfalls, provider limits, and security considerations.
- [ ] Consult the official Solidity ABI and Ethereum JSON-RPC documentation for exact rules.
