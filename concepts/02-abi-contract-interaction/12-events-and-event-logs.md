# Events and Event Logs in Ethereum: Communicating Contract Activity

> **Stage:** 02 — ABI and Contract Interaction  
> **Priority:** 1  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-17

## 1. Overview

A Solidity **event** is a declared interface for writing structured information into the EVM's logging system. A contract emits an event when an action occurs, and off-chain applications can read the resulting **event log** from transaction receipts or through Ethereum JSON-RPC log queries.

Events are commonly used to communicate activity such as token transfers, deposits, withdrawals, role changes, protocol configuration updates, and order creation. They allow frontends, indexers, analytics services, and monitoring systems to react to contract activity without repeatedly scanning every storage slot.

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.24;

    contract SimpleVault {
        event Deposited(address indexed account, uint256 amount);

        mapping(address => uint256) public credit;

        function deposit() external payable {
            require(msg.value > 0, "Zero deposit");
            credit[msg.sender] += msg.value;
            emit Deposited(msg.sender, msg.value);
        }
    }

The declaration defines the event's name and parameters. The statement with <code>emit</code> writes a log entry containing the event's encoded arguments, along with metadata such as the emitting contract's address.

![Diagram showing a Solidity contract declaring and emitting an event, creating a log with an address, topics, and data in a transaction receipt, which an off-chain indexer queries and decodes](https://raw.githubusercontent.com/0xSix9/Blockchain-Concepts-Learning-Roadmap/main/assets/diagrams/events-and-event-logs-flow.svg)

[Open the Events and Event Logs diagram directly](https://github.com/0xSix9/Blockchain-Concepts-Learning-Roadmap/blob/main/assets/diagrams/events-and-event-logs-flow.svg)

**Important distinction:** an event is a Solidity-level declaration and use site; an event log is the actual EVM log record created during execution. The log is observable off-chain, but contracts cannot read their own past logs directly from EVM execution.

## 2. Declaring and Emitting Events

An event is declared with the <code>event</code> keyword. To emit it from a function, use <code>emit</code> and provide the arguments in the declared order.

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.24;

    contract Marketplace {
        event ItemPurchased(
            address indexed buyer,
            uint256 indexed itemId,
            uint256 price
        );

        function recordPurchase(uint256 itemId) external payable {
            require(msg.value > 0, "Payment required");

            emit ItemPurchased(msg.sender, itemId, msg.value);
        }
    }

This example declares a purchase event with three parameters. The buyer and item identifier are marked indexed, while the price is stored in the log's data field.

Declaring or emitting an event does not automatically change contract state. In real application logic, the state update, payment validation, and event should describe the same successful operation. A common convention is to emit the event after the relevant state effects have been applied.

### Events and function execution

Emitting an event does not call an external listener, invoke a frontend callback on-chain, or return an event object to the caller. It executes the EVM logging operation. Off-chain software learns about the log by querying a node, watching subscriptions, or reading the transaction receipt.

A function can emit more than one event, and different functions can emit the same event declaration. For event-driven integrations, name events after the domain action they describe and include enough context for consumers to distinguish the operation.

## 3. What an Event Log Contains

An Ethereum log record includes three major components:

- **Address:** the address of the contract that emitted the log in the current execution context.
- **Topics:** a list of 32-byte values used for event identification and filtering.
- **Data:** ABI-encoded values for the event parameters that were not declared indexed.

Transaction receipts expose the logs produced by that transaction. Each log also appears in RPC responses with contextual metadata such as its block number, transaction hash, transaction index, and log index. Exact response fields can vary by RPC method and client.

For a standard non-anonymous event, the structure is conceptually:

    Log {
        address:  contract address,
        topics:   [event signature hash, indexed arguments...],
        data:     ABI-encoded non-indexed arguments
    }

The log is not self-describing. To interpret its topics and data correctly, a consumer generally needs the event definition from the contract ABI, including each parameter's type and whether it is indexed. The same raw bytes can be decoded incorrectly if a consumer supplies the wrong event schema.

### Where the logs are found

Logs are included in transaction receipts and are committed through Ethereum's block data structures. An application can retrieve a known transaction's receipt using JSON-RPC, or query matching logs across a range of blocks with <code>eth_getLogs</code>. WebSocket-based subscriptions can notify an application about matching new logs, subject to the node's subscription support and chain behavior.

A log's presence in a receipt indicates it was produced by that execution result. It does not by itself guarantee the block can never be reorganized out of the canonical chain. Applications that act on valuable events should consider block confirmations, finality, and reorganization handling.

## 4. The Event Signature Topic

For an ordinary non-anonymous event, <code>topics[0]</code> contains the Keccak-256 hash of the event's canonical signature.

For example:

    event Deposited(address indexed account, uint256 amount);

The canonical signature string is:

    Deposited(address,uint256)

The parameter names, the <code>indexed</code> keyword, spaces, and the event's return behavior are not included in that signature. The parameter types are written in their canonical ABI form. The hash of this signature identifies which event type the log claims to represent.

This signature topic is not the same as a function selector. A function selector uses the first four bytes of the Keccak-256 hash of a function signature; an event's default signature topic uses the full 32-byte hash.

### Example: decoding a standard event

Consider the declaration:

    event OwnershipTransferred(
        address indexed previousOwner,
        address indexed newOwner
    );

A typical log for this event contains:

- <code>address</code>: the contract that emitted the ownership change.
- <code>topics[0]</code>: the hash for <code>OwnershipTransferred(address,address)</code>.
- <code>topics[1]</code>: the encoded previous owner address.
- <code>topics[2]</code>: the encoded new owner address.
- <code>data</code>: empty, because there are no non-indexed parameters.

The precise bytes can be decoded with the matching ABI. Consumers should not infer the event's meaning from an event name copied from an untrusted source; use the expected contract address and the correct interface when validating logs.

### Anonymous events

Solidity also allows an event to be declared <code>anonymous</code>. For such an event, the usual event-signature hash is not automatically placed in <code>topics[0]</code>. An anonymous event can therefore use up to four indexed parameters rather than the three available alongside the signature topic in a standard event.

Anonymous events are less convenient to identify by signature alone, and can make logs easier to misinterpret if consumers expect an ordinary event. Use them only when their behavior is deliberate and the integration is designed to recognize the log correctly.

## 5. Indexed Parameters, Topics, and Data

The <code>indexed</code> keyword determines which event parameters are placed into topics and which are placed in the data field.

For a standard non-anonymous event, Ethereum reserves one topic for the event signature. Up to three parameters may be indexed, giving at most four topics total. An anonymous event has no automatic signature topic and can use up to four indexed parameters.

    event Transfer(
        address indexed from,
        address indexed to,
        uint256 amount
    );

In a standard event log for this declaration:

- Topic 0 is the hash of <code>Transfer(address,address,uint256)</code>.
- Topic 1 represents <code>from</code>.
- Topic 2 represents <code>to</code>.
- Data contains the ABI-encoded <code>amount</code>.

Indexed value types that fit within a topic are encoded into 32-byte topic words. Indexed arrays, strings, bytes, and other complex/reference values are represented by a Keccak hash of the specified event-parameter encoding rather than being stored as their full original value in the topic. That makes filtering by a known value possible, but it usually does not allow a consumer to recover the original dynamic value from the hash alone.

Non-indexed arguments are ABI-encoded together in the log's data field. They can be decoded when the consumer knows the event's ABI and has the raw data.

**Design trade-off:** indexing a parameter can make it more convenient and efficient to filter for a known value, but it uses a limited topic slot. Leaving a parameter non-indexed makes its value directly available in decoded data. Choose indexed fields based on realistic query patterns, not by marking every parameter indexed automatically.

The next Stage 02 topic, **Topics and Indexed Parameters**, goes deeper into encoding rules, topic filters, dynamic types, and filtering patterns; its dedicated article will be linked here when published.

## 6. Filtering and Consuming Logs Off-Chain

Applications commonly query logs by:

- **Contract address:** restrict results to logs emitted from a target contract address or set of addresses.
- **Block range:** scan a chosen range or continue from the last processed block.
- **Event-signature topic:** select the event type for standard non-anonymous events.
- **Indexed argument topics:** filter by known values such as a user address, token identifier, or order ID.

Ethereum JSON-RPC includes <code>eth_getLogs</code> for retrieving matching logs. Clients and libraries typically let developers provide an address, a block range, and a topics filter. The precise query format and available range limits depend on the provider.

Conceptually, an event consumer might:

1. Load the expected contract ABI and address.
2. Query the selected block range for matching log topics.
3. Decode each log with the correct event definition.
4. Process each event idempotently, recording enough metadata to avoid duplicate handling.
5. Continue monitoring new blocks and handle reorganizations according to the application's confirmation/finality policy.

An event-driven user interface can show newly deposited funds or a completed purchase after finding the relevant log. An indexer can build a database of trades, NFT transfers, or protocol actions. A monitoring service can alert when a sensitive configuration event appears.

Do not assume a WebSocket notification is permanently canonical just because it arrived first. Some providers may notify consumers that a previously delivered log was removed because its block was reorganized out. Reliable consumers should be able to roll back or reconcile affected records, and may use receipt verification, block hashes, confirmation depth, or chain finality depending on the value at risk.

### A log is not the contract's state

Events are useful records of activity, but they are not a replacement for reading contract state when the application needs the current authoritative value. A consumer may miss a notification while disconnected and need to backfill logs, and not every important value is necessarily emitted by an event.

Use logs to learn that a contract says an action happened; use state reads and appropriate chain validation when you need the contract's current value or a security-critical invariant.

## 7. Events, State Variables, and Return Values

These three mechanisms serve different purposes.

### State variables

State variables hold data used by the contract during EVM execution. Smart contracts can read their own storage according to Solidity and EVM rules. External applications usually query state through RPC calls, and a contract can enforce rules based on that state.

### Event logs

Logs record emitted information for off-chain consumers and filtering. Contracts cannot query historical logs from within their own execution, and event logs are not a general-purpose replacement for state storage. Events can be cheaper and more convenient for publishing history than keeping every historical record in storage, but an application should not emit an event instead of storing data that the contract itself must later use.

### Return values

A function's return values are delivered through the call's return data to its caller. They are not automatically persisted as transaction logs. An ordinary top-level transaction receipt does not contain the return value of the function as an event log; applications often inspect events or query state to learn the outcome after the transaction is mined.

For example, a function can both return a computed value to an external contract and emit an event for off-chain indexing. These communicate through distinct mechanisms with different consumers.

## 8. Reverts and the Lifetime of Logs

Logs are part of EVM execution state for purposes of success and rollback. If an execution path emits a log but then reverts, that log does not survive in the successful transaction receipt as a committed event from that reverted execution.

For example:

    function tryOperation() external {
        emit OperationStarted(msg.sender);

        require(false, "Operation failed");

        emit OperationCompleted(msg.sender);
    }

In this deliberately failing example, the entire call reverts. Neither emitted event remains as a committed log from the reverted call. This is important for interpreting attempted actions: off-chain systems should not treat a log emitted only before a reverted operation as a successful permanent event.

Nested calls require similar care. If a child call emits a log and then its execution is reverted, the child call's changes and logs are rolled back. If the parent catches the child failure and continues, the parent can still successfully emit its own event describing the failure or the fallback behavior. Logs only survive to the receipt if they belong to execution that was not reverted by the final result.

The receipt's overall status, the emitting contract address, the event's semantics, and the transaction's canonical-chain status should all be considered when building an indexer or analytics pipeline.

## 9. Event Design and Security Considerations

### Emit events for meaningful state transitions

Events should have clear names and consistent semantics. Common examples include:

- Deposits and withdrawals that update recorded balances.
- Ownership or role changes.
- Token transfers or approvals.
- Orders opened, filled, cancelled, or expired.
- Protocol parameter changes and upgrades.

For state-changing operations, document what the event means and when it is emitted. For example, a <code>Withdrawal</code> event should clearly indicate whether it means a withdrawal was requested, an accounting entry was cleared, or the transfer was actually completed.

### Do not treat events as authorization

Events do not execute permission checks or modify contract state on their own. Emitting <code>AdminChanged</code> does not make a caller an administrator. The contract must first validate the operation and perform the state transition, then emit an event that accurately reflects the successful result.

### Do not put secrets in logs

Event data is public to anyone able to inspect the chain. Marking a parameter indexed does not make it private; it merely moves it into a searchable topic. Do not emit private keys, passwords, access tokens, or other information that must remain confidential. Hashing a predictable secret into a topic is not necessarily safe because observers may be able to guess candidate values and compare hashes.

### Do not assume an event proves the whole business action

Any contract can emit its own event using its own address. A log with a familiar-looking signature is not proof that a trusted protocol emitted it. Integrations should validate the emitting address, expected chain, ABI, and relevant transaction/block context. A malicious contract can also define an event with the same signature as a legitimate event.

### Be deliberate about topic and gas trade-offs

Logs consume gas, and indexing changes the shape and usefulness of the log. Emit only useful details and use indexed fields for the parameters consumers are likely to filter by. Event logs are not free storage; excessive event emission can make transactions more expensive and increase data-processing costs for off-chain consumers.

## 10. Common Mistakes

1. **Forgetting <code>emit</code>.** A declaration alone does not create a log; execution must emit it.
2. **Assuming event data is readable on-chain.** Contracts cannot directly query historical logs during normal EVM execution.
3. **Confusing topics and data.** Indexed arguments go into topics; non-indexed parameters are ABI-encoded in data.
4. **Forgetting the signature topic.** Standard events normally use one of the four topic slots for the signature, leaving up to three indexed parameters.
5. **Assuming dynamic indexed values are stored in plain text.** Complex or dynamic indexed values are stored as hashes, not as their original full values.
6. **Treating emitted logs as successful actions even when execution reverts.** Reverted execution's logs are rolled back.
7. **Assuming a log notification is final forever.** Chain reorganizations may remove or replace logs from the canonical chain.
8. **Using events as a substitute for required contract storage.** Contracts cannot read historical events to enforce their own later rules.
9. **Trusting a familiar event signature without validating the emitter.** Verify the contract address and chain, not just the topic.
10. **Emitting sensitive data.** Events are public; indexing is a filter mechanism, not a privacy feature.

## 11. Key Takeaways

- Solidity events declare structured log formats; <code>emit</code> creates the log during execution.
- A log contains the emitting address, up to four topics, and ABI-encoded data.
- Standard non-anonymous events put a Keccak-256 hash of the canonical event signature into topic 0.
- Indexed parameters are placed in topics for filtering; non-indexed parameters are stored in the data field.
- Off-chain consumers query receipts or log RPC methods and decode logs using the correct event ABI.
- Logs from reverted execution are rolled back and do not persist as successful receipt logs.
- Events are public and useful for indexing, but they do not replace state storage, authorization, return values, or chain-finality handling.

## References

- [Solidity Documentation — Contracts: Events](https://docs.solidity.org/en/latest/contracts.html#events)
- [Solidity Documentation — ABI Specification: Events](https://docs.solidity.org/en/latest/abi-spec.html#events)
- [EIP-140 — REVERT instruction](https://eips.ethereum.org/EIPS/eip-140)
- Topics and Indexed Parameters (planned companion topic; dedicated article not yet published)
- [Function Dispatch](04-function-dispatch.md)
- [External Function Calls](05-external-function-calls.md)
- [Fallback Functions](07-fallback-functions.md)
- [Receive Functions](08-receive-functions.md)
- [State Mutability](10-mutability.md)
- [Payable Functions](11-payable-functions.md)

## Review Checklist

- [x] Explains event declarations, emit, and the event-log abstraction.
- [x] Describes log address, topics, data, ABI decoding, and the signature topic.
- [x] Explains the role of indexed parameters without duplicating the later deep-dive topic.
- [x] Covers log filtering, receipts, reorgs, and reverted execution.
- [x] Distinguishes events from state storage, return values, and access control.
- [x] Links official Solidity documentation, ABI specification, and EIP-140.
