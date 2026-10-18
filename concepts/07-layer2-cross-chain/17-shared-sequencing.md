# Shared Sequencing: One Ordering Layer for Multiple Rollups

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

![Shared sequencing architecture: users submit transactions for multiple rollups to one decentralized sequencer network. The network agrees on a common ordered stream and publishes data or commitments to a data availability layer. Each rollup filters its own transactions, executes them with its own state machine, and settles through its own proof and bridge contracts. Shared ordering can enable coordination, but atomic execution requires additional protocol logic.](../../assets/diagrams/17-shared-sequencing.svg)

## 1. Overview

**Shared sequencing** is an architecture in which one sequencing network orders transactions for multiple rollups instead of each rollup relying exclusively on its own separate sequencer. The network establishes a common order of submitted transactions or transaction bundles across participating rollups. Each rollup then processes the transactions relevant to its own execution environment.

The motivation is twofold. First, sharing a sequencing network can reduce dependence on a single rollup operator and spread transaction inclusion and ordering across a broader set of participants. Second, a common ordering context can help rollups coordinate activities such as cross-rollup trading, arbitrage, and message delivery, potentially reducing the fragmentation caused by independent sequencers that cannot see one another's pending transactions.

A shared sequencer usually does **not** need to execute every rollup's state machine. In a design known as *lazy sequencing*, it agrees on the order and inclusion of rollup transaction data while each rollup executes its own transactions according to its own rules. This keeps the shared sequencing layer more agnostic to individual rollup virtual machines and application logic.

Shared sequencing is not the same as a shared execution layer, shared settlement contract, or universal cross-chain bridge. A common order can provide the groundwork for coordination, but it does not automatically make transactions on different rollups atomic. To guarantee that an operation across multiple chains either fully succeeds or fully fails, the participating rollups need additional execution, commitment, and failure-handling rules.

## 2. Why independent sequencers create fragmentation

A typical rollup uses a sequencer to receive transactions, decide their ordering, build L2 blocks, and submit batches or commitments to a settlement layer. Many systems historically began with a sequencer operated by the rollup team because this is relatively simple to deploy and can provide fast user-facing confirmations.

When several rollups operate separate sequencing systems, three problems appear.

### Ordering is isolated

A sequencer normally sees its own rollup's pending transactions. It may not know that a transaction on another rollup is about to change a price, consume liquidity, or create a message that another application depends on. Cross-rollup applications therefore have limited ability to coordinate order across those domains.

### Sequencing power is concentrated

If one operator has exclusive control over inclusion and ordering, it can potentially censor transactions, delay them, or extract ordering value. The exact power depends on the rollup's forced-inclusion and escape mechanisms, but a centralized sequencing service is a meaningful component in the trust model.

### Cross-rollup interactions become asynchronous

If Rollup A and Rollup B have different sequencers, users often submit one transaction and wait for its outcome before submitting the next one. A transaction may succeed on A while the intended follow-up on B fails, creating partial outcomes, capital inefficiency, or the need for a bridge and separate recovery policy.

Decentralizing each rollup's sequencer can reduce reliance on a single operator, but it does not by itself create a shared global ordering. Shared sequencing is intended to address coordination across rollups as well as the concentration of sequencing power.

## 3. Core components of a shared sequencing system

The architecture varies by project, but several roles commonly appear.

### User transaction intake

Users submit transactions or order-flow bundles for the rollup they want to use. A submission must identify the destination rollup or namespace so that the appropriate rollup can later extract its own transactions.

A wallet or rollup front end may submit directly to the shared sequencing network. Some designs provide a helper service, such as a composer or transaction router, that wraps ordinary rollup transactions and pays or abstracts sequencing fees on the user's behalf.

### Sequencer validators and consensus

A network of sequencing nodes proposes and agrees on a common ordered set of rollup transactions. The consensus mechanism determines who can propose blocks, how peers vote, when an ordering decision becomes final under that sequencing network's rules, and what happens during network failure or validator equivocation.

Different systems use different consensus designs. Astria's documented architecture uses CometBFT for its sequencing network, while Espresso's sequencing and confirmation network uses HotShot. These are distinct implementations and should not be treated as a single protocol.

### Shared ordered stream

The sequencing network produces blocks or commitments containing transactions tagged for their target rollups. The shared order can place transactions from several rollups in one sequence. Each rollup's node identifies the records relevant to it and derives its own input stream.

A sequence record or commitment proves what the sequencing layer ordered, not necessarily that the destination rollup's execution will succeed. The rollup still validates transaction format, account state, signatures, nonces, gas rules, and contract conditions according to its own protocol.

### Data availability and retrieval

Rollup nodes must obtain the data they need to derive their blocks. A sequencing network may make data available through its own network, publish to another data availability layer, or use commitments with a separate retrieval path. These choices are important: agreement on an order is not useful if rollup nodes cannot retrieve the underlying transaction data.

For example, Astria documents a design in which its sequencing network orders transactions for multiple rollups, and sequenced data is published through Celestia for broader availability. Rollup-specific Conductor components retrieve and filter relevant data for their rollups.

### Rollup-specific execution and settlement

Each rollup uses the common ordered stream as an input, but executes with its own state transition function. It still needs an execution client, state derivation logic, data availability assumptions, and a settlement or verification mechanism. Depending on the rollup, state transition validity may be proven with validity proofs or established through an optimistic challenge process.

Shared sequencing does not merge these rollups into one state machine.

## 4. End-to-end transaction flow

A generic shared sequencing flow looks like this:

1. **Submit:** A user submits an L2 transaction or bundle and identifies its target rollup.
2. **Ingest:** A composer, gateway, or direct client sends the transaction to the shared sequencer's mempool or submission interface.
3. **Propose:** A sequencing proposer collects pending transactions for one or more rollups and builds a candidate shared block.
4. **Reach consensus:** The shared sequencing network agrees on the order and inclusion of the candidate data under its consensus rules.
5. **Publish or expose data:** The ordered transaction stream or a commitment to it is made available to rollup nodes through the specified DA and retrieval path.
6. **Filter:** Each rollup extracts the transactions or transaction bundles associated with its own identifier.
7. **Execute:** Rollup nodes execute their transaction subset against that rollup's state. Two rollups can have different virtual machines, fee models, account systems, and application logic.
8. **Settle:** Each rollup follows its own proof, challenge, and settlement rules. Sequencing finality is not automatically the same as L1 settlement finality.
9. **Coordinate cross-rollup effects:** If a user operation spans rollups, the application or protocol needs rules for message verification, conditional execution, failure, timeout, and recovery.

The most important boundary is between steps 4 and 7: the shared sequencer can establish a common order before each rollup finishes executing its part. If a transaction later fails because a contract condition is no longer true or a dependent state change did not happen, a common order alone does not reverse work already committed on another rollup.

## 5. What shared sequencing can improve

### A common ordering context

Because transactions from participating rollups are ordered by one sequencing network, builders can reason about their relative ordering across those rollups. This can be useful for cross-rollup arbitrage, coordinated liquidity operations, and protocols that need consistent ordering assumptions.

A shared order can also reduce ambiguity about which transaction came first from the sequencing layer's perspective. However, each rollup must consume and execute the ordered transactions correctly for that order to have the intended effect.

### Reduced dependence on a dedicated sequencer per rollup

A rollup that integrates with a shared network may not need to maintain its own exclusive sequencing consensus. Depending on the integration, it can outsource ordering to a decentralized validator set and rely on the shared network's consensus and availability design.

That can reduce duplicated infrastructure across rollups, but it introduces a new shared dependency. If the shared network stalls, censors data, changes its rules, or loses availability, several connected rollups may be affected together.

### Better cross-rollup coordination

A common sequencing layer provides a mechanism for coordinating transaction inclusion across rollups. In appropriately designed systems, a user can submit related transactions or a bundle to multiple rollups and obtain shared commitments about their order or inclusion.

This is the foundation for stronger interoperability, but applications must inspect exactly what is guaranteed. A system may guarantee only that the transactions are placed into a common sequence. Another system may add conditional bundle inclusion, state-aware preconfirmations, or protocol logic that attempts to make multi-rollup operations atomic.

### Potentially more efficient block building

A shared network can aggregate order flow from multiple rollups and give specialized builders a broader view of transaction demand. It may allow better bundle construction or fee aggregation. Whether this benefits users depends on fee policy, competition, network overhead, and which participants can access the ordering market.

### Shared developer infrastructure

Rollups can integrate with a common sequencing API, consensus layer, or data publication pipeline rather than building every component independently. This can reduce integration work, but it is not free: rollups must implement derivation rules, authenticate sequencing commitments, monitor the shared network, and design failure handling for the new dependency.

## 6. Atomic inclusion is not atomic execution

This is the most important security distinction in shared sequencing.

**Atomic inclusion** means a group of transactions across rollups is either included according to the sequencing system's all-or-nothing rule or is not included at all. The exact guarantee depends on the sequencer's protocol and the way it defines a bundle.

**Atomic execution** means the intended state changes across all participating rollups happen together: either every required sub-transaction succeeds, or the system reaches a defined state in which none of the operation's effects remain. This requires control over execution outcomes and failure recovery, not just the order of transactions.

Consider a two-step DeFi operation:

- On Rollup A, a user swaps Token X for Token Y.
- On Rollup B, the user deposits Token Y into a lending market.

The shared sequencer can order both transactions together. But Rollup A's swap could execute successfully while Rollup B's deposit reverts because collateral limits, contract state, or a deadline has changed. If the first transaction's state change is already committed and there is no cross-rollup rollback protocol, the user's overall intent is only partially completed.

A shared sequence is therefore a useful primitive, not the whole atomicity implementation. Stronger guarantees may require one or more of the following:

- Conditional bundle inclusion based on explicit execution constraints.
- A prepare/commit protocol, escrow, or state locks to prevent partial effects.
- Shared validity sequencing that binds cross-rollup message commitments and defines what happens if one side does not finalize.
- Application-level compensation or refund rules when full atomicity is impossible.
- A protocol that coordinates finality and failure proofs across every participating rollup.

Even these mechanisms have trade-offs in complexity, liveness, capital locking, and dependence on the shared coordinator. A rollup integration should explicitly describe whether it offers ordered inclusion, conditional inclusion, synchronized execution, or true atomic execution.

## 7. Shared sequencing versus related architectures

### Independent sequencers

Each rollup has its own sequencing service or set. This gives teams direct control over ordering and policy, but the ordering context is fragmented. Interoperability generally needs asynchronous messaging or separate coordination protocols.

### Decentralized per-rollup sequencers

A rollup uses multiple nodes and consensus to decentralize its own ordering. This can reduce dependence on one sequencer, but the sequencer set is still specific to that rollup. Decentralization and sharing are separate properties: a sequencer can be decentralized but not shared.

### Shared sequencer

A common network orders transactions for multiple rollups. It may be decentralized or permissioned; the words shared and decentralized are not interchangeable. The design must specify the membership model, consensus, censorship guarantees, data availability, and finality.

### Based rollups

A based rollup uses Ethereum L1 proposers and block production to drive its sequencing. Shared sequencing generally refers to an additional sequencing network or coordination layer that handles multiple rollups. These architectures can pursue similar goals but rely on different sources of ordering and have different latency, trust, and incentive assumptions.

### Shared settlement

A shared settlement layer is responsible for verifying or settling state transitions from multiple rollups. That is a separate architectural function. A shared sequencer might serve rollups settling to Ethereum, another L1, or different settlement systems; shared sequencing does not imply shared settlement.

### Interoperability and messaging protocols

Cross-chain messaging transmits authenticated messages or state claims between domains. It can enable cross-rollup applications even without shared sequencing. Shared sequencing may simplify coordination and establish common ordering commitments, but message authentication, finality, destination execution, and failure handling are still needed.

## 8. Example systems: Astria and Espresso

Two commonly discussed approaches illustrate how shared sequencing can be implemented differently.

### Astria: sequencing with rollup-specific execution

Astria's documented architecture provides a decentralized sequencing layer for multiple rollups. Its sequencing network orders transactions tagged with a rollup identifier and can remain agnostic to the target rollup's transaction format and state transition function. Its design describes lazy sequencing: the shared layer orders data, while rollup nodes execute the relevant transactions themselves.

Astria also documents a separation of components such as Composer, Relayer, and Conductor. The Composer helps submit user transactions, the Relayer moves sequencing data toward the data availability layer, and a rollup-specific Conductor obtains and filters data for the relevant rollup. In the documented stack, Celestia is used for broader data availability of sequenced data. These components demonstrate why ordering, availability, derivation, and execution must be considered as separate responsibilities.

### Espresso: shared ordering and fast confirmations

Espresso describes a sequencing or confirmation layer for rollups using its HotShot consensus protocol. Rollups can integrate by assigning identifiers or namespaces to their transactions and obtaining inclusion evidence from the sequencing ledger. Rollup nodes then extract their own data, execute their own state transitions, and settle using the applicable rollup verification mechanism.

The exact guarantees depend on the integration and the confirmation stage. Fast sequencing-layer confirmation should not be casually equated with settlement finality on Ethereum or another L1. Integration code must establish what the commitment proves, how data can be obtained, and what finality guarantees are relied on for bridge operations.

These projects are examples of the architecture, not proof that every shared sequencer provides identical security or atomic composability. Each should be assessed through its current specifications, consensus rules, DA arrangement, rollup integration, and failure behavior.

## 9. Security and failure modes

### Sequencer consensus failure

If the shared network loses consensus, stops producing blocks, or has too few active participants, connected rollups may be unable to sequence new transactions through that layer. A design needs recovery and fallback rules, including whether rollups can switch sequencing paths and how conflicting histories are handled.

### Censorship or selective ordering

A validator set or proposer may exclude some users, prefer certain transactions, or manipulate ordering. Decentralization and rotating proposers can mitigate exclusive control, but they do not automatically eliminate censorship or MEV. The protocol should specify submission access, ordering policy, penalties, and whether users have alternative inclusion paths.

### Data unavailable after sequencing

A consensus certificate may establish that the sequencer agreed on an ordered block, but rollup nodes still need the transaction data to execute it. A shared sequencing design must clarify whether data is directly distributed, published to a DA layer, or retrievable through another mechanism. Ordering without recoverable data can leave rollups unable to derive or execute their state.

### Cross-rollup partial execution

Even if the sequencer includes multiple transactions together, execution on one rollup can fail while execution on another succeeds. Atomic bundle inclusion is not enough unless rollups and the application define conditional execution, rollback, escrow, or compensation.

### Reorganizations and finality mismatches

A shared sequencing layer may provide its own notion of finality, while each rollup has an L1 settlement process. A bridge or app that treats a preliminary sequencing commitment as final settlement may create risk. A safe design maps the states of sequencing confirmation, data availability, L2 execution, proof/challenge completion, and L1 finality.

### MEV concentration and privacy

A shared sequencer gains a broad view of submitted order flow across rollups. This can improve bundle construction but can also concentrate ordering power. Private mempools, encrypted order flow, fair ordering, or auction mechanisms may reduce certain forms of exploitation, but each has its own assumptions and latency or complexity trade-offs.

### Governance and upgrade authority

Changes to consensus rules, validator membership, rollup integrations, data availability providers, or the sequencing-token economy can affect every connected rollup. Shared infrastructure can create shared failure domains. Integrators should identify who can upgrade software or contracts, how emergency changes are enacted, and how rollups can exit the network if terms or security assumptions change.

## 10. Developer evaluation checklist

Before integrating a rollup with a shared sequencer, ask:

- Which sequencing network is used, and is its validator set permissionless, permissioned, or stake-weighted?
- What consensus algorithm establishes transaction order, and what does a sequencing commitment actually certify?
- Does the network sequence raw transactions, opaque data, commitments, or pre-built blocks?
- Is rollup-specific data published to a DA layer, and how does a rollup node retrieve and verify it?
- Can rollup nodes deterministically filter transactions for their own namespace and derive blocks from the shared stream?
- What happens if the shared sequencer halts or the DA layer is unavailable?
- Is there a safe fallback to another sequencer or L1 forced inclusion path, and how are conflicting histories prevented?
- Does the integration guarantee only shared order, atomic inclusion, conditional bundle inclusion, or atomic execution?
- How are execution failures on one rollup handled when another rollup's transaction has succeeded?
- How do the sequencing layer's fast confirmations relate to the rollup's proof acceptance and L1 finality?
- Who can censor or reorder transactions, and what recourse do users have?
- How are MEV, sequencing fees, validator rewards, and bundle priorities allocated?
- Are message proofs and bridge contracts independent of the shared sequencer's untrusted claims?
- What governance or upgrade authority can change the guarantees on which the rollup relies?

## 11. Key takeaways

- Shared sequencing orders transactions for multiple rollups through a common sequencing network.
- It can reduce duplicated sequencing infrastructure and create a common ordering context for coordination across rollups.
- A shared sequencer can be decentralized or permissioned; those are separate design properties.
- In lazy sequencing, the shared network orders data while each rollup executes its own state machine.
- Shared ordering can enable stronger cross-rollup coordination, but atomic inclusion does not automatically produce atomic execution.
- Data availability, rollup validity, settlement finality, bridge authentication, and failure recovery remain separate security responsibilities.
- Astria and Espresso illustrate different implementations; their consensus, DA, and confirmation guarantees should be evaluated separately.
- Shared sequencers can reduce fragmentation but can also become shared failure and governance dependencies for every participating rollup.

## References

1. [Astria Docs — Introduction](https://docs.astria.org/overview/introduction)
2. [Astria Docs — The Astria Sequencing Layer](https://docs.astria.org/overview/components/the-astria-sequencing-layer)
3. [Astria Docs — How Rollups Work](https://docs.astria.org/overview/how-rollups-work)
4. [Astria Docs — Transaction Flow](https://docs.astria.org/overview/transaction-flow)
5. [Astria Docs — Composer and Order Flow](https://docs.astria.org/overview/components/composer)
6. [Espresso Systems — Shared Sequencing: Defragmenting the L2 Rollup Ecosystem](https://hackmd.io/@EspressoSystems/SharedSequencing)
7. [Espresso Network — ZK Rollup Integration](https://github.com/EspressoSystems/espresso-network/blob/main/doc/zk-integration.md)
8. [CRATE — Cross-Rollup Atomic Transaction Execution](https://arxiv.org/abs/2502.04659)
9. [Trireme — Shared Sequencing and Atomic Execution](https://www.trireme.com/research/shared-sequencing)
10. [Astria — Source Code](https://github.com/astriaorg/astria)
