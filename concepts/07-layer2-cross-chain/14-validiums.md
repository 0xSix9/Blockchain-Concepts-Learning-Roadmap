# Validiums: Validity Proofs with Off-Chain Data Availability

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

![Validium architecture: an L2 operator executes transactions and publishes state commitments and validity proofs to Ethereum, while transaction data is stored by an external data availability layer or committee. The on-chain verifier checks state validity, but users still depend on the off-chain availability design to recover data and exit safely.](../../assets/diagrams/14-validiums.svg)

## 1. Overview

A **validium** is a Layer 2 scaling design that uses a validity proof to convince Ethereum that a proposed state transition follows the system's rules, while keeping the data needed to reconstruct that state off Ethereum Mainnet. The proof and state commitment are verified or recorded on-chain, but the underlying state-diff data is stored by a separate data availability mechanism, such as a Data Availability Committee (DAC) or another external data availability network.

This separation creates the core trade-off. Validiums can reduce the amount of data they pay to publish on Ethereum, allowing higher throughput and lower data-publication costs. But they introduce an additional trust and availability assumption: users must be able to obtain the data needed to reconstruct their balances and prove their ownership or exit if the operator stops cooperating.

A validity proof can prove that an update was computed correctly according to the circuit or state-transition rules. It cannot, by itself, reveal a dataset that the operator or every data holder withholds. **Correct execution and data availability are separate security properties.**

This article focuses on the architecture, trust assumptions, withdrawal implications, and comparison with ZK-rollups, Optimiums, and Volitions. It is intended for smart-contract developers evaluating L2 security models, bridges, and application deployment choices.

## 2. The basic architecture

A typical validium has both off-chain services and on-chain contracts.

### Off-chain execution operator

The operator receives user transactions, orders them, executes them, and constructs a new L2 state. It computes a new state root or another commitment and prepares a state transition for proof generation. Its service may include a sequencer, execution engine, state database, and prover coordination.

### Validity prover

A prover produces a cryptographic proof that the new state follows the system's rules and is consistent with the previous state. The proof system may be based on a SNARK, STARK, or another validity-proof construction. The exact scheme is implementation-specific; the word validium describes the data-availability architecture, not one proof system.

### Data availability layer or committee

The transaction data or state differences required to reconstruct the L2 state are stored off-chain. In a DAC-based design, committee members receive the batch data, verify that it is available to them, store copies, and sign an availability attestation. The committee's signatures are checked by an on-chain contract or state-update process before the new state is accepted.

Other designs may rely on a separate DA network with its own consensus, sampling, staking, or economic security. These mechanisms differ in their assumptions; “off-chain data availability” does not automatically mean the project has a DAC.

### Ethereum settlement contracts

The L1 contract stores accepted state commitments and verifies validity proofs. It may also check an availability certificate, such as a threshold of DAC signatures, before allowing a state update. The bridge contract holds or controls assets associated with the L2 and applies the system's deposit and withdrawal rules.

Exact components depend on the implementation, but the security design must answer two different questions: is the new state valid, and can users obtain the data necessary to act on that state?

## 3. A validium state transition step by step

A simplified state update can be described as follows:

1. **Users submit transactions.** The operator receives orders, transfers, trades, or other L2 actions.
2. **The operator executes a batch.** It applies the transactions to the existing L2 state and calculates the proposed new state commitment.
3. **The batch data is distributed off-chain.** The data required to reconstruct the relevant state changes is sent to the DA service, committee members, or other designated storage participants.
4. **Availability is attested.** In a committee-based design, members check the data and sign a message that identifies the batch or state update. A signature should be bound to the specific batch and commitment, not be a generic promise that some data exists.
5. **The prover constructs a validity proof.** The proof establishes that the state transition follows the protocol's rules, using the required public inputs and private witness data.
6. **The L1 contract checks the update.** It verifies the proof and any required availability certificate, then records or accepts the new state commitment.
7. **Users interact with the accepted state.** Their ability to independently reconstruct the state and withdraw in an operator-failure scenario depends on access to the off-chain data and the escape mechanisms the system provides.

The ordering and contract logic vary by implementation. A system may coordinate data availability and proof generation before submitting the update. The essential point is that proof verification alone does not prove that every user can retrieve the data later.

## 4. Why validity proofs do not solve data availability

Suppose an account tree contains Alice's balance and the operator publishes only the new root of that tree. A validity proof can establish that the transition from the previous root to the new root followed the protocol rules. However, if Alice later needs a Merkle proof for her balance and the operator refuses to provide the account tree or the required state differences, the root alone may not give her enough information to construct that proof.

A cryptographic commitment is a compact commitment to data, not a copy of the data. Likewise, the proof establishes a statement about the transition but does not automatically make all private witness or state-diff data public.

In an on-chain-data rollup, the data needed to reconstruct the state is published on Ethereum, making it available under Ethereum's data availability assumptions. In a validium, that data is kept externally. The L1 can continue to reject invalid state transitions while still lacking the information needed for users to reconstruct the latest state independently.

This is the central security distinction:
- **Validity protection:** the system rejects state transitions that fail its proof or verification rules.
- **Availability protection:** the system makes the required state data obtainable to independent users.
- **Exit protection:** the system gives users a way to prove ownership and withdraw even when the operator is malicious or offline.

These mechanisms overlap in the user experience, but none should be assumed to replace the others.

## 5. Data Availability Committees (DACs)

A **Data Availability Committee** is a group of entities responsible for receiving, checking, storing, and serving the off-chain data associated with batches or state updates. In a common model, committee members sign a batch-specific availability claim, and the L1 contract accepts the batch only if enough required signatures are present.

This arrangement can be practical because a small group of storage providers can maintain redundant copies without publishing every state update on Ethereum. It also makes operational responsibility clear: members must run services, keep persistent data, monitor storage integrity, and answer retrieval requests.

However, a DAC adds trust assumptions outside Ethereum's own consensus and data availability. The critical questions include:

- How many committee members exist, and who selects them?
- What threshold of signatures is needed to accept a state update?
- Are any members mandatory signers?
- Are members independent organizations, or do they share owners, infrastructure, cloud providers, jurisdictions, or governance?
- Do signatures prove only that members received data, or are there stronger monitoring, replication, and retrievability procedures?
- What incentives or penalties exist for withholding data?
- Can committee membership or signature thresholds be changed by an upgrade key or governance process?
- What does the L1 contract do if data becomes unavailable after an otherwise valid state update?

A threshold certificate is not the same as proof that every user can always retrieve the data. If enough signers collude, fail, or lose their copies, the system may accept a state whose data is later difficult or impossible to retrieve. The practical risk depends on the threshold, actual independence, durable replication, and the design of the escape path.

## 6. Withdrawal and operator-failure scenarios

Withdrawal security is where the availability model becomes concrete. In a non-custodial system, users need a way to recover their assets without relying solely on the operator's voluntary cooperation.

A validium may define an escape mechanism where the application contract stops accepting new state updates and users can withdraw by supplying a Merkle proof against the latest accepted state. Such a path is useful only if users can acquire the state data and produce a valid proof. If the data holders withhold the state differences or the relevant tree data, a user may know that a balance exists but be unable to construct the proof required by the contract.

A design review should therefore trace at least these scenarios:

**Operator halts but DA providers remain honest.** Can users obtain the latest data, build proofs, and exit directly through L1? Is there a documented escape hatch that does not require operator approval?

**Operator and some DA members collude.** Does the contract require an availability threshold that prevents state updates without enough independent signers? Can a quorum still sign a batch and then refuse to serve the data later?

**Data is acknowledged but subsequently lost.** Do multiple independently managed copies exist? Are backups checked? Are data retention and recovery responsibilities enforceable, and do they cover the period when users may still need proofs?

**The L1 verifier or bridge is upgradeable.** Can an administrator change verification rules, alter the withdrawal path, or bypass the availability check? The proof system may be sound while an upgrade key still has powerful permissions over the contracts.

**The operator disappears before an update is finalized.** What is the latest accepted state on L1? What data does a user need to withdraw from that state, and who can supply it?

These questions are more important than a label such as “ZK-based.” The proof system's mathematical security and the data availability and bridge controls must be evaluated separately.

## 7. Validium vs ZK-rollup vs Optimium

The distinction between these designs is best understood by separating state validation from data availability.

### ZK-rollup

A ZK-rollup publishes the data needed to reconstruct the L2 state to Ethereum and uses validity proofs to verify state transitions. Its data-availability guarantee is based on Ethereum's on-chain data availability assumptions. This generally costs more L1 data fees than a validium, but users do not have to trust an external committee as the only source of the data needed for state reconstruction.

The proof type alone does not define whether a system is a rollup or validium. The location and security assumptions of the data are essential.

### Validium

A validium uses validity proofs, but the data necessary to reconstruct the state is made available off Ethereum, often using a DAC or another DA network. This can reduce L1 data costs and may enable additional design options, but adds external availability assumptions. Under the broader industry's common terminology, a validium is often described as a validity-proof-based L2 with off-chain data availability.

### Optimium

An Optimium uses an optimistic or fraud-proof-based state validation approach while relying on external data availability. Because data is not published to Ethereum, its security also depends on an external availability mechanism. The distinction from a validium is the state validation mechanism: validity proofs versus fraud proofs or an optimistic challenge process.

Terminology differs across projects and trackers, and actual systems can have hybrid configurations. Always inspect the verification contract, data publication path, committee or DA layer, upgrade authority, and withdrawal mechanism rather than relying solely on the marketing name.

## 8. Volition: allowing different data-availability choices

A **Volition** design allows an application or user to choose between an on-chain-data rollup mode and a validium mode, often at an asset, account, or transaction level supported by the implementation. That means the user can select a higher-trust-minimized option that publishes the relevant state data on L1, or a lower-cost mode whose data is handled externally.

StarkEx documentation, for example, describes ZK-Rollup, Validium, and Volition data-availability modes for supported applications. In that model, the application can offer both data-availability modes with different vault arrangements. This is a useful architectural example, but it should not be assumed that every project calling itself a Volition uses the same implementation.

Choice comes with complexity. The UI needs to communicate the security and cost difference, bridge and state contracts must preserve correct mappings, and users need to understand whether funds in different modes have different availability assumptions. A cheaper setting is not simply a cosmetic fee option; it can change who must be trusted to provide data.

## 9. Advantages and limitations

### Potential advantages

- **Lower L1 data costs.** The system does not need to publish all state-diff data to Ethereum, reducing one major cost component for large or data-heavy applications.
- **Higher throughput within the design.** Off-chain data publication can reduce the constraint imposed by L1 data capacity, though execution, proving, network, and DA limits still apply.
- **Flexible data architecture.** A project may tailor storage, replication, and availability operations to its workload.
- **Potential confidentiality benefits.** Keeping application data off public L1 can reduce immediate public visibility. This is not the same as cryptographic privacy: committee members or storage providers may see the data, and other disclosures may reveal it.
- **Useful for data-heavy applications.** Trading, gaming, and application-specific systems may value throughput and cost efficiency enough to accept an external DA model, provided the trade-off is transparent.

### Limitations and risks

- **External trust assumptions.** Users rely on a committee, DA network, or storage design beyond Ethereum.
- **Data withholding.** A valid state root and proof do not help users reconstruct missing state data.
- **Operational complexity.** Durable storage, replication, monitoring, member churn, and recovery procedures become core parts of security.
- **More complex risk evaluation.** A well-designed validity proof does not remove committee or bridge risks.
- **Potentially weaker censorship or escape guarantees.** If a system cannot supply data or accept independent withdrawal proofs, users may be unable to exit promptly.
- **Confidentiality is not guaranteed.** Off-chain storage does not by itself encrypt data or prevent committee members from reading it.

## 10. How to evaluate a validium as a developer

Before integrating an application or bridge with a validium, make a security checklist that goes beyond “does it use a validity proof?”

1. **Proof system:** What does the proof actually attest to? Which public inputs are bound to the previous state, new state, batch, and chain context?
2. **Data format:** Which transactions, state differences, account-tree changes, or other data are needed to reconstruct the state and generate withdrawal proofs?
3. **Data holders:** Who keeps the data? Are they a permissioned DAC, a public DA network, or multiple independent services?
4. **Availability certificate:** What exactly is signed? Is the message bound to a unique batch ID and state commitment? What threshold is enforced by the verifier contract?
5. **Retrievability:** Can users access data directly or only through the operator's API? Are copies and independent retrieval services tested regularly?
6. **Escape hatch:** Can the contract enter a mode where users withdraw without operator cooperation? What information and proofs must each user provide?
7. **Failure threshold:** How many committee members can fail or collude before data can be withheld despite a batch being accepted?
8. **Bridge security:** Does the bridge use the verified state root and correct withdrawal proofs? Can upgrade keys bypass or change those rules?
9. **Governance and upgrades:** Who controls the verifier, bridge, committee membership, threshold, and emergency controls?
10. **User communication:** Does the product explain its data availability assumptions, withdrawal process, and residual risks in terms users can understand?

The key task is to trace the full user journey: deposit, transaction execution, batch acceptance, data retrieval, and withdrawal when the operator or DA provider is unavailable. Security exists across that whole path, not only in the proof verifier.

## 11. Key takeaways

- A validium verifies state transitions with validity proofs but stores the data needed to reconstruct state off Ethereum.
- The proof can show that a state transition is valid; it cannot force withheld data to become available.
- DAC signatures or external DA consensus provide additional availability assumptions that must be evaluated separately from proof soundness.
- Withdrawal safety depends on having the data and an independent route to construct and submit the required exit proof.
- ZK-rollups and validiums may both use validity proofs; where the data is published is a critical architectural difference.
- Optimiums typically combine external data availability with optimistic or fraud-proof-based validation.
- Volition can let users or applications choose between on-chain and external data availability, but the modes have different trust and cost profiles.
- “Lower cost” and “validity-proven” do not automatically mean the system has rollup-equivalent data availability.

## References

1. [Ethereum.org — Validium](https://ethereum.org/developers/docs/scaling/validium/)
2. [StarkEx Documentation — Data Availability](https://docs.starkware.co/starkex/con_data_availability.html)
3. [StarkEx Documentation — High-Level Overview](https://docs.starkware.co/starkex/overview.html)
4. [StarkEx Data Availability Committee Reference Implementation](https://github.com/starkware-libs/starkex-data-availability-committee)
5. [L2BEAT — FAQ and Layer 2 Definitions](https://l2beat.com/faq)
6. [L2BEAT — Validiums and Optimiums Risk Analysis](https://l2beat.com/scaling/risk?tab=validiumsAndOptimiums)
7. [L2BEAT — Data Availability Risk Framework](https://forum.l2beat.com/t/the-data-availability-risk-framework/318)
