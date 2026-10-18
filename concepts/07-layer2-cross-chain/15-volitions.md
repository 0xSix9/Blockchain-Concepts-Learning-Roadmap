# Volitions: Choosing Between On-Chain and Off-Chain Data Availability

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

![Volition architecture: a shared L2 validity-proof system supports two data-availability paths. Rollup-mode state differences are published to Ethereum, while Validium-mode data is handled by an external availability committee or network. The application maps balances and withdrawals to the corresponding data tree and makes the trust trade-off visible to users.](../../assets/diagrams/15-volitions.svg)

## 1. Overview

A **Volition** is a Layer 2 architecture that combines on-chain data availability, as used by a rollup, with off-chain data availability, as used by a validium. Instead of forcing an application to choose one model for every account or supported asset, a Volition-enabled system can let users or applications select a data-availability mode for the supported part of their activity.

The idea is to make an architectural trade-off configurable. Publishing the data needed to reconstruct state on Ethereum gives users the availability guarantees of the underlying L1 data-publication model, but it consumes L1 data capacity and costs fees. Keeping that data off-chain can reduce data-publication cost and increase throughput, but users must trust or rely on the external mechanism to provide the data when it is needed.

A Volition does not remove this trade-off. It exposes both choices in one application or protocol. Depending on implementation, a user may choose a mode for a vault, balance, supported asset, or other unit of state. The exact granularity is not universal; integrations must check the design of the specific system.

StarkEx is a documented example. Its Volition mode supports both rollup and Validium vaults for supported spot-trading applications, with separate vault ranges and corresponding data-availability handling. A user can choose the relevant mode for assets under that application's supported model. That example illustrates the concept, but it should not be taken to mean every Volition offers the same interface or per-transaction switch.

## 2. The core design problem

A Layer 2 system needs to execute transactions, update balances or contract state, and provide enough evidence for other parties to verify state transitions. When a user needs to exit, the user may also need data to construct a proof of ownership against the latest accepted state.

There are two distinct data-availability choices:

- **On-chain data availability:** the state changes required to reconstruct the L2 state are published to Ethereum. This is the common data model for a rollup. It costs L1 data fees, but users do not need to rely solely on an external committee for the relevant state data.
- **Off-chain data availability:** the data needed to reconstruct state is kept outside Ethereum, for example by a Data Availability Committee (DAC) or a separate DA network. This can reduce L1 publication costs, but it adds availability assumptions beyond Ethereum's own data-publication mechanism.

A Volition combines both choices in a shared application architecture. It does not mean that a single piece of data is simultaneously treated as both fully on-chain and off-chain. Instead, the implementation records which data-availability path applies to each supported balance, vault, account, or other state unit and ensures that the state update and exit rules use the correct path.

## 3. The architecture at a glance

A Volition typically needs several interacting components.

### L2 operator and execution service

The operator receives user transactions, orders them, executes them, and computes the resulting state. It constructs state differences, updates the relevant state trees, and coordinates proof generation and data publication. As with other L2 designs, the operator and sequencer are part of the transaction-processing path, but they should not be treated as the sole source of truth for withdrawals.

### Validity prover

A prover generates a proof that the state transition follows the system's rules. The proof is verified by an L1 contract or another explicitly trusted verification component. The proof system may be SNARK-based, STARK-based, or another validity-proof design.

The proof is about correctness of the transition. It does not automatically make off-chain data retrievable. Availability controls remain necessary for every state path whose data is not published to Ethereum.

### Two data-availability paths

The **rollup path** publishes the relevant state differences or data needed for reconstruction to Ethereum. An independent user can recover the information from the L1 data, subject to Ethereum's own availability and retention model.

The **Validium path** sends the corresponding data to an external mechanism, such as a DAC. A DAC-based implementation can require committee members to store the data and sign an availability approval bound to the batch. An external DA network may use a different consensus or sampling design.

### State trees and vault mapping

Implementations need a way to identify which availability model applies to a balance or state unit. In the StarkEx Spot Volition design, there are distinct vault ranges and separate state-tree handling for rollup and Validium vaults. Updates and escape or withdrawal paths need to resolve to the correct tree.

This mapping is a security-critical detail. If the UI labels an asset as rollup-backed but the relevant balance is actually held in a Validium vault, users may misunderstand which availability assumptions apply. The implementation must keep user-facing mode information consistent with the state and contract logic.

### L1 settlement and bridge contracts

The on-chain contracts verify proofs and update state commitments, enforce any required availability approvals, and control deposits or withdrawals. Their logic must distinguish the data-availability path of the relevant state. The precise arrangement varies by application, so contract verification and system documentation matter more than the Volition label alone.

## 4. How a Volition transaction flows

The following is a generic model. A real implementation may define selection at the asset, vault, account, or another supported level.

1. **Choose a supported mode.** When creating or moving a supported balance, the user or application selects the rollup-data path or the Validium-data path. The system must communicate the implications clearly.
2. **Submit the transaction.** The user sends an action to the L2 service, such as depositing, transferring, trading, or moving funds between supported vaults.
3. **Resolve the target state path.** The application determines which tree or vault corresponds to the selected availability mode. If the operation moves value between modes, it must update both sides consistently.
4. **Execute the batch.** The operator calculates the resulting state changes and prepares the relevant state commitments and validity proof.
5. **Publish or distribute data according to the mode.** Data associated with rollup-mode state is published on-chain. Data for Validium-mode state is delivered to the designated committee or external DA mechanism, which follows its own availability process.
6. **Verify and accept the state update.** The L1 contract verifies the validity proof and applies any required availability-certificate checks before accepting the update.
7. **Reflect the mode in balances and exits.** The application shows users which mode their funds are in and ensures withdrawal proofs are generated against the correct state tree.
8. **Handle mode changes explicitly.** If a system allows a user to transfer an asset from one data-availability mode to another, that move is a state transition with defined costs, proof handling, and data-publication obligations. It is not merely a display-setting change.

The critical implementation requirement is consistent accounting across the two state paths. The system must not create duplicated claims, omit a state update, or let a balance appear in one tree while its transfer or withdrawal is authorized against another.

## 5. What actually changes when a user chooses a mode?

The mode selection changes the location and security assumptions of the data needed to reconstruct state; it does not necessarily change the validity-proof system or the L2 execution model.

### Rollup mode

The data needed to reconstruct the relevant state changes is published on Ethereum. This increases L1 data costs, but gives users a path to recover the relevant state information from L1 data without depending exclusively on an external DA committee. A forced exit still requires correct contract logic and the proof material needed to establish ownership, but the required state differences should be recoverable from the on-chain data.

### Validium mode

The data remains off Ethereum and is supplied by the external DA arrangement. Users can benefit from lower L1 data-publication costs, but a forced exit may depend on access to the external data or an escape process supported by the DA participants. The validity proof does not eliminate this dependency.

### Moving between modes

A Volition may let users transfer a balance from one vault or data-availability path to another. The system must prove that the move is valid, debit the source representation, credit the destination representation, and publish or distribute the data required by both paths. Depending on the protocol, the transfer may incur different fees or processing steps.

The same asset can therefore have different availability assumptions depending on the mode or vault in which it is represented. Applications should avoid presenting these as equivalent just because the token symbol or user interface looks identical.

## 6. Security: configurable trade-offs, not configurable cryptography

A Volition is not normally a choice between “valid” and “invalid” transactions. Both paths can use the same validity-proof system and enforce the same application transition rules. The main difference is where the state data is made available and what recovery assumptions follow from that choice.

Key risks include:

**Data withholding.** In Validium mode, a committee or external DA layer may fail to supply data. A valid proof and state commitment do not reconstruct a withheld balance tree.

**Incorrect mode mapping.** If contracts, indexers, or front ends associate a balance with the wrong state tree, users may receive misleading information about how to prove ownership or withdraw.

**Inconsistent cross-mode transfers.** A move between rollup and Validium representations must debit and credit exactly once. Errors in state-tree updates or asynchronous processing can lead to stuck funds, duplicated accounting, or invalid withdrawal proofs.

**Availability changes after acceptance.** A DAC can acknowledge data and later lose it or refuse to serve it. Security review must examine both the acceptance certificate and how long data is retained, replicated, and retrievable.

**Bridge and verifier upgrades.** Administrator keys or upgrade paths may change proof verification, availability checks, state-tree rules, or the escape hatch. Validity proofs do not protect users against every privileged contract change.

**UI and product misrepresentation.** A mode label is useful only if it tracks the actual state. Users need to understand whether a balance depends on Ethereum-published data or external availability providers, and what an emergency withdrawal would require.

The right question is not “Does this application use ZK proofs?” but “For this specific balance, which data is on-chain, who provides any off-chain data, and how does the user exit if those providers stop cooperating?”

## 7. Volition versus rollups and Validiums

### Compared with a ZK-rollup

A ZK-rollup publishes state-reconstruction data on Ethereum and verifies state transitions with validity proofs. A Volition offers a rollup mode but can also offer another mode that keeps data off-chain. The rollup mode generally incurs more L1 data cost than Validium mode, but its availability assumptions are more directly tied to Ethereum.

### Compared with a Validium

A Validium uses an off-chain data-availability design for the state covered by that mode. A Volition includes that kind of path but pairs it with an on-chain-data path in the same application. A pure Validium deployment need not offer users the rollup alternative.

### Compared with an Optimium

An Optimium generally combines external data availability with optimistic or fraud-proof-based state validation. A Volition is usually discussed in the context of combining rollup and Validium data-availability modes around a validity-proof-based system. These labels describe different architectural dimensions: state validation and data availability must both be inspected.

### Compared with a generic hybrid DA system

Not every protocol that uses multiple data sources is a Volition. The term commonly refers to an application architecture in which a supported user, asset, vault, or state unit can choose between on-chain rollup availability and off-chain Validium availability. The granularity, accounting model, and switching rules depend on implementation.

## 8. Performance, cost, and privacy implications

Volition can let an application match data-publication costs to its use cases. A data-intensive activity with many state changes might prefer the lower-cost off-chain path, while a user who places greater value on independent data recovery might choose the on-chain path. Whether that choice is actually cheaper depends on L1 fees, batching, proof costs, DA-provider fees, and the implementation's charging model.

It is important not to overstate the privacy benefit. Keeping data off public L1 can limit public visibility of state details, but it does not automatically provide cryptographic privacy. The operator and the entities holding off-chain data may see the information. Confidentiality requires additional controls, such as encryption, access restrictions, privacy-preserving computation, or a cryptographic protocol designed for that purpose.

Likewise, more throughput does not automatically mean better security. The architecture shifts some costs and assumptions away from Ethereum's L1 data publication and toward external availability services. Users and integrators should understand both the economic benefit and the additional trust surface.

## 9. Example: choosing a vault mode

Imagine a trading application supports two modes for the same class of asset.

- Alice prioritizes independent data recovery. She places her balance in a rollup-mode vault, so the relevant state differences are published on Ethereum. She pays the cost associated with that publication path.
- Bob prioritizes lower fees and accepts the external availability assumptions. He places a supported balance in a Validium-mode vault, whose data is provided by the committee or configured DA network.
- Alice or Bob wants to change modes. The application must perform a defined transfer or state update from one tree to the other. It cannot simply alter a metadata flag, because the source and destination representations and their data-publication rules differ.
- The operator becomes unavailable. Alice's exit path relies on the state data published on-chain and on the application's escape mechanism. Bob's exit may require the Validium data holders to provide the information needed for his proof. The precise outcome depends on each path's contract design and recovery procedures.

This example is schematic, not a statement that every Volition uses two separate balance trees. It illustrates why the mode and its consequences must be traceable from user interface to state commitment and withdrawal contract.

## 10. Developer checklist

When evaluating or integrating a Volition, review these questions:

- What is the exact unit of choice: asset, vault, account, transaction, or application configuration?
- How does the contract encode the selected mode, and can the selection be altered after a balance is created?
- Which state differences are published to Ethereum, and which are sent off-chain?
- What data is required to reconstruct the state tree and produce an exit proof for each mode?
- If there is a DAC, what threshold signs each state update, and are signatures bound to the correct batch and state commitment?
- If an external DA network is used, what consensus, sampling, staking, or trust assumptions does it make?
- Are the two state paths accounted for separately and consistently in transfers, deposits, bridge messages, and withdrawals?
- Does a cross-mode transfer debit and credit the appropriate representations atomically or through a safe, replay-protected transition?
- What happens if data is unavailable after the L1 accepts a valid state update?
- Is there an escape hatch that lets users withdraw without operator cooperation, and what proofs or data does it require?
- Who can upgrade the verifier, bridge, mode mapping, or escape logic?
- Does the UI clearly tell users which availability model applies to their actual balance?
- Have the security, fee, and retrieval implications been explained without implying that off-chain data is automatically private?

## 11. Key takeaways

- A Volition combines rollup-style on-chain data availability and Validium-style off-chain data availability within one application or protocol.
- It exposes a choice among data-availability trade-offs; it does not remove those trade-offs.
- Exact selection granularity varies by implementation. StarkEx Spot Volition is a documented example with distinct rollup and Validium vault ranges.
- Validity proofs can protect state-transition correctness, but do not make off-chain data retrievable.
- Correct state-tree mapping, cross-mode accounting, and withdrawal handling are essential implementation details.
- Off-chain data can reduce L1 publication costs, but may add availability trust assumptions and does not by itself provide cryptographic privacy.
- Developers should evaluate each mode's data path, bridge contracts, recovery procedure, DA assumptions, and upgrade authority.

## References

1. [Ethereum.org — Validium and Volitions](https://ethereum.org/developers/docs/scaling/validium/)
2. [StarkEx Documentation — Data Availability Modes](https://docs.starkware.co/starkex/con_data_availability.html)
3. [StarkEx Documentation — High-Level Overview](https://docs.starkware.co/starkex/overview.html)
4. [StarkEx Spot Trading 4.5 Release Notes — Volition](https://docs.starkware.co/starkex/release_notes/4.5.html)
5. [L2BEAT — Data Availability Risk Framework](https://forum.l2beat.com/t/the-data-availability-risk-framework/318)
