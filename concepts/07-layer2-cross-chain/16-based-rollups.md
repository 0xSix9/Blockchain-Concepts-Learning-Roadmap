# Based Rollups: Ethereum Layer 1 as the Sequencer

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

![Based rollup architecture: users submit L2 transactions, builders and searchers prepare an L2 block or bundle, and the Ethereum L1 proposer can include its sequencing data in an L1 block. The rollup still uses its own execution and settlement rules, while data availability and validity or fraud proofs determine its broader security model.](../../assets/diagrams/16-based-rollups.svg)

## 1. Overview

A **based rollup** is a rollup whose transaction sequencing is driven by the base Layer 1 (L1), rather than being controlled exclusively by a separate L2 sequencer. In the common Ethereum definition, the next Ethereum block proposer can, together with builders and searchers, permissionlessly include the next rollup block as part of an Ethereum L1 block. The L1's block-production process therefore helps determine the ordering of L2 transactions.

The core idea is to reuse Ethereum's proposer and block-building ecosystem instead of operating a separate sequencer set as the primary source of transaction ordering. If L2 sequencing is driven by Ethereum's permissionless proposer market, a rollup can reduce reliance on a dedicated sequencing authority and align some of its sequencing incentives with Ethereum's validators and builders.

A based rollup is still a rollup. L2 transactions are executed according to the rollup's execution rules, and the rollup needs a settlement design that determines how state updates are accepted. Depending on its architecture, it may use validity proofs or optimistic fraud proofs, and it must provide the data required by its data-availability model. **Based sequencing describes who drives transaction ordering; it does not, by itself, specify the proof system or data-availability guarantee.**

## 2. What problem does based sequencing address?

Many rollups use a dedicated sequencer or a restricted sequencer set. A sequencer receives L2 transactions, orders them, provides users with fast acknowledgements, constructs L2 blocks, and submits batches or commitments to L1. This can deliver a smooth user experience, but it creates a privileged point in the transaction-ordering path.

Depending on its design, a centralized sequencer may be able to delay transactions, exclude users, choose transaction order for its own benefit, or temporarily stop producing blocks. A rollup can mitigate these risks through forced-inclusion mechanisms, escape hatches, decentralized sequencing, or other controls, but those mechanisms have their own delays and complexity.

Based rollups explore another approach: route the sequencing role through Ethereum's existing block production. Ethereum proposers already have the right to propose L1 blocks, and builders and searchers already compete to construct profitable block contents. A based rollup can let this infrastructure also build and order L2 batches.

That approach can reduce the need for a separate sequencer consensus system and create a stronger relationship between L1 liveness and the availability of new L2 sequencing opportunities. It does not mean that every L2 transaction is automatically included, that censorship is impossible, or that users receive instant finality. The exact outcome still depends on the rollup's submission rules, the L1 inclusion path, the proposer and builder market, and the L1 consensus protocol.

## 3. The core definition

The foundational definition, introduced in Ethereum research by Justin Drake, describes a rollup as based or L1-sequenced when the next L1 proposer may permissionlessly include the next rollup block in the next L1 block, potentially working with L1 searchers and builders.

Three parts are important:

- **L1-driven ordering:** the L1 proposer and the block-building pipeline determine which sequencing data is included and therefore drive the order of accepted L2 blocks.
- **Permissionless opportunity:** the design does not require a single exclusive L2 sequencer to be the only actor permitted to submit the next L2 block or sequencing payload, though the actual protocol rules determine who can construct and submit valid data.
- **Rollup rules remain in force:** L2 state transitions, proof verification, bridge accounting, data publication, and upgrade controls still need to be secured separately.

A based rollup does not necessarily require an Ethereum validator to execute L2 transactions itself. Often, specialized builders or searchers construct the L2 block or an L1 transaction bundle that carries L2 sequencing data. The L1 proposer chooses whether to include it in its L1 block, subject to the applicable protocol and block-building process. This preserves the division between sequencing, execution, proving, and settlement.

## 4. A simplified block-production flow

A generic based rollup flow can be understood as follows. Different projects have different transaction formats, proving pipelines, and submission rules.

1. **Users submit L2 transactions.** Transactions enter the rollup's public or private transaction-propagation channels.
2. **Searchers and builders collect transactions.** One or more participants construct a candidate L2 block or batch, often optimizing ordering, execution, and the economics of including the corresponding L1 transaction.
3. **The candidate is encoded for Ethereum.** The builder prepares the transaction or bundle that submits the rollup sequencing data through the rollup's L1 entry point.
4. **An L1 proposer selects an L1 block.** The proposer, commonly working through a block-building marketplace or related infrastructure, can include the based rollup payload. The rollup's sequencing is driven by the proposer-selected L1 block production process.
5. **The L1 block is propagated and accepted.** Ethereum nodes validate the L1 block. The rollup observes the accepted L1 sequencing data under its derivation and fork-choice rules.
6. **The L2 state is derived and executed.** L2 nodes process the included transactions according to the rollup's execution rules and derive the corresponding state.
7. **The rollup establishes settlement.** Depending on its proof design, state updates are accepted through the validity-proof or fraud-proof system and the required data-availability conditions. Sequencing inclusion is not the same as proof verification or final settlement.
8. **Users observe stronger inclusion evidence over time.** L1 inclusion and subsequent L1 consensus progress provide evidence that sequencing data has entered the rollup's ordering path. The exact finality and preconfirmation guarantees depend on the protocol.

The sequence separates three events that user interfaces often blur: a sequencer acknowledgement, inclusion of sequencing data in an L1 block, and the rollup state becoming finalized under its settlement rules. These events can occur at different times and provide different guarantees.

## 5. Based rollup versus a dedicated sequencer

### Dedicated sequencer model

A dedicated sequencer offers an operator or a defined set of operators privileged access to order L2 transactions. It can provide fast soft confirmations, impose custom transaction policies, run its own ordering rules, and coordinate execution and batch submission. The model may be simple to operate, but security depends on the sequencer's decentralization, availability, censorship policy, and escape mechanisms.

### Based sequencing model

A based rollup delegates its primary sequencing driver to the L1 block-production process. The next L1 proposer can include an L2 block or sequencing payload, typically prepared by builders. This reuses the L1's proposer selection and economic infrastructure, and can reduce a rollup's need for an independent sequencing consensus and special sequencer-signature validation.

However, it also inherits constraints from L1 block production. The rollup cannot assume that its own preferred sequencer can instantly confirm every transaction regardless of the L1 schedule. Fast preconfirmations, strict first-come-first-served ordering, and application-specific transaction policies may require additional mechanisms.

### Shared sequencer is not the same thing

A shared sequencer serves more than one rollup and attempts to coordinate ordering across them. A based rollup, by contrast, uses L1 proposers to drive its sequencing. These concepts can overlap in some designs or be combined with additional coordination, but a based rollup does not automatically provide cross-rollup atomic composability.

For example, several based rollups may each submit their sequencing payload through the same L1 block. That can create useful ordering opportunities, but whether applications can rely on cross-rollup transactions executing atomically depends on additional protocols, sequencing commitments, and failure handling. Sharing an L1 proposer is not, alone, a complete shared-sequencing protocol.

## 6. What security properties can improve?

### Liveness linked to Ethereum

If Ethereum continues producing blocks and the rollup permits eligible participants to submit its sequencing data, based sequencing can reduce dependence on one separate sequencer's uptime. The rollup's ability to progress is more directly linked to the L1's block-production process.

This is not a guarantee of zero downtime. Ethereum can experience missed slots, network partitions, reorgs, or congestion. The rollup may also have its own constraints in candidate construction, transaction admission, proving, or state transition processing.

### Reduced exclusive sequencing power

A design that permits multiple independent builders or searchers to prepare candidate L2 payloads can limit the power of a single exclusive L2 sequencer. Proposer competition may give more participants the opportunity to influence ordering and inclusion. The degree of permissionlessness depends on the contracts, transaction-submission rules, builder ecosystem, and economic incentives; it should be evaluated from implementation rather than inferred from the name.

### Incentive alignment with L1

Based sequencing can route some rollup-originated MEV and block-building opportunities through Ethereum's proposer and builder market. This aligns certain economic activities with the L1 and may reduce the need for a rollup-specific sequencing token or a separate proof-of-stake sequencing committee.

The trade-off is that the rollup may give up revenue it could otherwise capture from its own sequencing process. A protocol can still charge L2 base fees or other fees, but who receives execution fees, ordering value, or MEV depends on the implementation.

### Simpler sequencing trust model

By reusing Ethereum's sequencing process, a based rollup may avoid some components of a separate sequencer set, such as dedicated sequencer consensus or signature checks for a privileged sequencer. But the system still needs secure L1 contracts, correct L2 execution, proof verification, data publication, bridge controls, and safe upgrade permissions. “Less custom sequencing machinery” does not mean “no security surface.”

## 7. What based sequencing does not guarantee

Based rollup terminology is specifically about sequencing. Developers and users should not assume it automatically grants every property of Ethereum L1.

- **It does not automatically provide validity proofs.** The rollup still needs to define how state transitions are accepted, using its chosen validity or fraud-proof design.
- **It does not automatically guarantee data availability.** The rollup must still specify whether transaction data is posted to Ethereum, blobs, or a separate DA system. An L1-sequenced system that uses external DA still inherits that external availability risk.
- **It does not eliminate MEV.** Searchers and builders still compete to order transactions and construct profitable blocks. Based sequencing changes where and how some MEV is captured rather than making it disappear.
- **It does not prevent every form of censorship.** Proposers and builders may omit transactions, and users still depend on the rollup's transaction propagation and inclusion rules. Inclusion-list or forced-inclusion mechanisms can complement the design, but those are additional mechanisms.
- **It does not imply instant finality.** L1 inclusion, L1 finality, rollup proof acceptance, and bridge withdrawal finality are different milestones.
- **It does not make L2 execution free.** L1 posting costs, L2 execution fees, proving costs, and block-building economics remain.
- **It does not automatically create cross-rollup atomicity.** Shared timing or proposer participation is not a complete interoperability mechanism.
- **It does not guarantee faster user confirmations.** Centralized sequencers can give rapid soft confirmations. A based design that waits for L1 inclusion may need preconfirmation arrangements to match this user experience.

These distinctions are important when comparing architectures. A rollup can be based and validity-proven, based and optimistic, or combine based sequencing with other modular design choices, as long as the specific architecture supports that combination.

## 8. Trade-offs: liveness, latency, MEV, and revenue

### Inclusion latency and preconfirmations

Ethereum L1 follows its slot schedule, so sequencing through L1 can tie progress to L1 timing and block availability. Dedicated sequencers often provide near-instant soft confirmations, before L1 inclusion or final settlement. A based rollup may need **based preconfirmations** or other credible commitment schemes if the application requires a rapid user acknowledgement.

A preconfirmation is not identical to finalized inclusion. Its guarantee depends on the actor making the commitment, its collateral or penalty system, the scope of the promise, and how the protocol handles a broken promise. Integrators should communicate whether a UI is showing a soft preconfirmation, L1 inclusion, or settlement finality.

### MEV distribution

A sequencer that controls transaction ordering can extract value from ordering opportunities and may use some of it to fund its operation. In based sequencing, L1 builders, searchers, and proposers participate in the transaction-ordering pipeline, so some of the resulting MEV can accrue at the base layer rather than to the rollup operator.

This could strengthen L1 economic alignment, but it may change the rollup's revenue model and the quality of user-facing services. The result is not guaranteed to be cheaper for all users: L1 fees, builder competition, the rollup's own fees, and transaction demand still matter.

### Ordering policy

A centralized sequencer can implement a simple first-come-first-served rule or other custom policy. In a based system, the L1 block-building market may reorder candidate transactions according to its inclusion and profit objectives. If a rollup promises strict ordering guarantees, it needs an enforceable specification that fits the L1 block-building model.

### Reorgs and settlement

An L1 reorganization can alter which sequencing transactions are canonical, and thus may require the L2 derivation pipeline to follow the rollup's reorg handling rules. The system must define when L2 applications can treat a transaction as included, when its result is provisional, and what happens if the relevant L1 block is reorged.

The rollup's validity or fraud-proof system adds another dimension. An L2 block being sequenced through L1 does not alone prove its state transition is correct. A safe design specifies both sequencing and settlement finality.

## 9. How based rollups relate to PBS

**Proposer-Builder Separation (PBS)** is an L1 block-production design in which the proposer and block builder have distinct roles. In current Ethereum infrastructure, many validators outsource sophisticated execution-payload construction to external builders through out-of-protocol systems. PBS research explores more protocol-integrated separation and the guarantees that make those roles safer and more censorship-resistant.

Based rollups can reuse the proposer-builder ecosystem: specialized builders and searchers can prepare an L2 sequencing payload while the L1 proposer determines which L1 block to propose. PBS-friendly pipelines may make this interaction more structured and reduce some coordination problems.

However, based rollups do not require a single specific finalized PBS protocol as part of their definition. PBS designs are evolving, and the status of specific EIPs can change. As of 2026-10-18, EIP-7732 is in Last Call (Standards Track: Core), with a Last Call deadline of November 1, 2026. It describes enshrined PBS and is useful background for understanding the separation of proposer and builder duties, but it is not synonymous with based rollups, nor should its Last Call status be mistaken for deployment on Mainnet or as a mandatory dependency for every based-rollup implementation.

An important distinction for smart-contract developers is that the builder generally prepares the payload and the proposer chooses whether to include it. The L1 proposer does not necessarily execute the L2 transactions itself. The rollup contracts, clients, and proof system still determine whether state updates are accepted.

## 10. Example: a trade on a based rollup

Imagine a decentralized exchange deployed on a based rollup.

1. Alice submits a swap transaction to the L2 transaction network.
2. A searcher or builder collects her transaction along with other pending L2 transactions and constructs a candidate L2 batch.
3. The builder creates the L1 submission that carries the rollup sequencing data. The candidate's position in the L1 block-building market may depend on fees and other bundle economics.
4. An Ethereum proposer includes the submission in an L1 block.
5. The rollup's nodes derive and execute the L2 block according to its rules. Alice's swap is now part of the L2 ordering path, subject to the rollup's defined inclusion and reorg rules.
6. The rollup's settlement system verifies the resulting state transition through its validity proof or optimistic challenge process.
7. The application displays the appropriate status: preconfirmed, included in L1, proven or challenge-complete, and finally settled according to its design.

This example shows why transaction status needs precise naming. A preconfirmation can be useful for user experience, but it is not interchangeable with L1 inclusion or finalized settlement. Apps should communicate the guarantee attached to each status.

## 11. Developer checklist

When analyzing a based rollup, inspect the full system rather than sequencing alone:

- Who may construct a candidate L2 block, and who may submit its sequencing payload?
- Does the next Ethereum proposer determine the order of L2 blocks, or is there still an exclusive sequencer with privileged control?
- How does the L1 block builder learn about rollup transactions and include them?
- What happens to transactions that are not selected by a proposer or are repeatedly omitted by builders?
- Are there forced-inclusion or inclusion-list mechanisms, and what are their timeout and censorship guarantees?
- What kind of preconfirmation does the user receive, who signs or guarantees it, and what happens if it is broken?
- Is L2 data published on Ethereum, including blobs or calldata, or does it depend on a separate DA layer?
- Which validity-proof or fraud-proof mechanism determines state-transition correctness?
- How are L1 reorgs reflected in the rollup's derivation, execution, and bridge accounting?
- When can a transaction be considered final for the application and for cross-chain withdrawals?
- Who captures L2 MEV and sequencing revenue? Does the model align with the rollup's sustainability?
- Which contracts or upgrade keys can change sequencing rules, proof verification, or escape mechanisms?
- Does the implementation provide genuine permissionless sequencing, or only reuse some L1 builder infrastructure?

## 12. Key takeaways

- A based rollup is an L2 rollup whose sequencing is driven by Ethereum L1 proposers and block production.
- Builders and searchers can prepare L2 blocks or sequencing payloads; the L1 proposer can include them in an L1 block.
- Based sequencing can reduce dependence on a dedicated sequencer and reuse L1 liveness and economic infrastructure, but does not make censorship or outages impossible.
- Sequencing, execution, data availability, proof verification, and final settlement are separate concerns.
- Based sequencing may change who captures MEV and how transactions are ordered; fast preconfirmations and strict first-come-first-served guarantees may need additional designs.
- PBS is related infrastructure, not a synonym for based rollups and not necessarily a mandatory finalized dependency.
- To evaluate a based rollup, inspect its submission path, L1 inclusion rules, DA design, proof system, reorg handling, withdrawal mechanism, and upgrade authority.

## References

1. [Ethereum Research — Based Rollups: Superpowers from L1 Sequencing](https://ethresear.ch/t/based-rollups-superpowers-from-l1-sequencing/15016)
2. [Fabric Docs — Fully Based Rollups](https://eth-fabric.github.io/website/education/composability/fully-based)
3. [Taiko — Based Rollup FAQ](https://www.paragraph.com/@taiko-labs/based-rollup-faq)
4. [Taiko Alethia — Protocol Repository](https://github.com/taikoxyz/taiko-mono)
5. [Ethereum.org — Proposer-Builder Separation](https://ethereum.org/roadmap/pbs/)
6. [EIP-7732 — Enshrined Proposer-Builder Separation](https://eips.ethereum.org/EIPS/eip-7732)
7. [Ethereum Foundation — How L1 and L2s Can Build the Strongest Possible Ethereum](https://blog.ethereum.org/2026/03/23/l1-l2-ethereum)
8. [Ethereum.org — Optimistic Rollups](https://ethereum.org/developers/docs/scaling/optimistic-rollups/)
9. [Ethereum.org — Zero-Knowledge Rollups](https://ethereum.org/developers/docs/scaling/zk-rollups/)
