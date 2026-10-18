# Fraud Proofs: Challenging Invalid Rollup State Transitions

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

## 1. Overview

A **fraud proof** is evidence used to challenge a claim that a rollup's execution or resulting state is correct. In an optimistic rollup, the protocol does not require a cryptographic validity proof for every proposed state transition up front. Instead, it provisionally accepts state claims under defined rules and gives independent participants an opportunity to dispute an invalid claim.

The term **fault proof** is also widely used. It is often more precise: the protocol establishes that a computation or claim is faulty, without needing to prove that the proposer intended to deceive anyone. The names and exact mechanisms differ among rollups, so the implementation's specification matters more than the label.

![Fraud proofs: from a proposed output to an interactive dispute and a one-step verification](../../assets/diagrams/01-fraud-proofs.svg)

In an optimistic rollup, transactions execute on Layer 2 (L2), while the rollup publishes the data needed to reconstruct its history and posts commitments or assertions to Ethereum Layer 1 (L1). An assertion may include an output root representing a particular L2 state. If someone believes the claimed result is wrong, the dispute protocol can determine which claim is consistent with the rollup's rules.

Fraud proofs are therefore part of a rollup's **state-transition security model**. They do not mean that every L2 transaction is independently proven valid on L1 when it is submitted.

## 2. Why Fraud Proofs Matter to a Developer

A rollup can execute transactions quickly while still needing a mechanism to resolve disagreement about its state. Without a meaningful challenge process, a malicious or faulty operator might submit an incorrect state commitment and attempt to use it to authorize a withdrawal.

Fraud proofs are important for three reasons:

- **Correctness:** they give independent participants a protocol-defined way to challenge an invalid state claim.
- **Withdrawal safety:** bridge contracts can rely on accepted outputs only after the rollup's proof, dispute, and finalization conditions are met.
- **Permissionless monitoring:** where the protocol permits open participation, independent nodes can recompute the rollup state and challenge an assertion without trusting the proposer.

A fraud-proof mechanism does not automatically make a rollup fully trustless. Its security also depends on transaction data being available, challengers being able to act before deadlines, the dispute contracts and verifier being correct, and administrative or upgrade powers not bypassing the process.

## 3. The Basic Model: Claim, Recompute, Dispute

An optimistic rollup generally follows this conceptual model:

1. **Publish inputs and a claim.** The rollup publishes transaction data or other protocol inputs and proposes an output or state commitment. The exact commitment format depends on the rollup.
2. **Reconstruct and execute.** Independent nodes use the protocol's data and prior state to replay L2 transactions according to the rollup's deterministic state-transition rules.
3. **Compare results.** A challenger finds that the recomputed result does not match the proposed claim.
4. **Open a dispute.** Within the applicable challenge window, the challenger starts the protocol's dispute procedure and submits any required bond or evidence.
5. **Resolve the claim.** The dispute game or verification mechanism determines which side's claim follows the rollup's rules. The winning claim can then be treated according to the rollup's finalization rules.

The on-chain contract does not necessarily execute the entire batch of L2 transactions again. Modern interactive fault-proof designs aim to narrow the disagreement until the L1 verifier needs to check only a small part of the computation.

### A simple state-root example

Suppose a rollup starts from an accepted state root `R0`, processes batch `B`, and its proposer claims that the resulting root is `R1`.

An independent node replays `B` from the correct starting state and computes `R1'`. If `R1' != R1`, the node has detected a disagreement and may have grounds to challenge the proposal.

This mismatch is **not, by itself, an on-chain fraud proof**. The challenger must follow the specific protocol, supply the required information, and win the dispute under the rollup's verification rules. The state root also needs to commit to the intended state representation; comparing unrelated or incorrectly anchored roots proves nothing.

## 4. Interactive Fault-Proof Games

One approach is **multi-round interactive proving**. Rather than asking Ethereum to re-execute a large L2 batch, the challenger and defender narrow the disagreement through successive claims.

A simplified game works as follows:

1. The defender stands behind the original claim; the challenger asserts that it is incorrect.
2. The disputed computation is represented as a trace or a sequence of state transitions.
3. The parties make protocol-defined attack and defense moves that narrow the range of disagreement, often by repeatedly splitting the trace into smaller sections.
4. When the disagreement has been reduced to a single instruction or another small base case, an on-chain virtual-machine (VM) verifier checks the relevant step using the required state and witness data.
5. The result of that verification determines the game outcome under the contract's rules.

The exact game tree, move timing, maximum depth, step semantics, and resolution rules are protocol-specific. Some designs use different proof systems or dispute-game constructions, so the interactive bisection model should not be assumed to describe every rollup.

### Example: the OP Stack approach

The OP Stack fault-proof specifications describe a stack with several distinct components:

- **Program:** computes a result from committed rollup inputs.
- **VM:** supports verification of the program's execution trace, including an individual disputed step.
- **Interactive dispute game:** lets participants narrow the disagreement until the VM can resolve the base case.
- **Preimage oracle:** provides authenticated input data needed to reproduce the relevant computation.

These components illustrate why a fault proof is more than a single hash submitted to a contract. The protocol must define what inputs are trusted, how the computation is reproduced, what constitutes a valid step, and how disagreement is resolved. Other rollups may use different architectures.

## 5. Bonds, Deadlines, and Finalization

Many dispute systems use bonds, clocks, and explicit resolution states to discourage spam and make participants respond in time. The precise incentives and outcomes are implementation-specific.

A typical design may require a proposer or challenger to lock collateral. If a claim is shown to be wrong, the protocol can reject it and apply the relevant bond rules. If a participant fails to make a required move before a deadline, the game may resolve against that participant. These outcomes must follow the dispute contract's formal rules; they are not universal properties of all fraud-proof systems.

Keep these stages separate:

- **L2 execution:** a transaction is processed by the L2 execution environment.
- **L1 data inclusion:** relevant batch data is included on Ethereum.
- **Output proposal:** a commitment is published for a particular L2 state.
- **Dispute resolution:** an assertion is challenged and the game resolves, if a dispute occurs.
- **Withdrawal finalization:** a bridge permits an L1 withdrawal only after its own proof and timing conditions are satisfied.

A fast sequencer confirmation is not equivalent to finality for an L1 withdrawal. Applications should describe which milestone their interface is reporting.

## 6. Data Availability Is a Prerequisite

Fraud proofs require challengers to obtain enough information to independently reproduce and dispute a claimed computation. If transaction data is withheld, an honest participant may be unable to determine whether the proposed state is correct or construct the evidence required by the dispute protocol.

This is one reason an optimistic rollup that publishes its data to Ethereum has a different trust model from a system that relies on an external data-availability committee. The latter may use fraud-proof or validity-proof logic for correctness, but it adds separate assumptions about access to data.

Do not conflate these questions:

- **Execution correctness:** does the claimed state follow from the specified state-transition rules?
- **Data availability:** can independent participants obtain the inputs needed to reconstruct that state?
- **Censorship resistance:** can a user force a transaction into the chain despite a sequencer refusing to include it?
- **Settlement and finalization:** when do protocol rules permit another contract, such as a bridge, to rely on the claim?

A fraud-proof mechanism addresses the first question. A secure system must also handle the others.

## 7. Common Mistakes and Security Considerations

### Assuming a challenge mechanism is automatically permissionless

Check who can create claims, who can challenge them, whether permissions or allowlists apply, and whether an independent participant can realistically complete every required step before the deadline.

### Ignoring the honest-challenger assumption

An optimistic design typically depends on at least one capable, honest party detecting an invalid claim and being able to challenge it in time. Incentives can help, but they do not replace access to data, a live dispute path, and correct verification logic.

### Treating all fraud-proof implementations as identical

Single-round replay, multi-round bisection games, and proof-assisted dispute games have different on-chain costs and failure modes. Check the target rollup's current specification instead of copying timeout values, bond rules, or VM assumptions from another network.

### Trusting the root without checking what it commits to

A state or output root is a commitment, not proof that the computation producing it was valid. The protocol must bind the claim to the correct chain, state-transition rules, inputs, and relevant block or message context.

### Overlooking upgrades and emergency controls

An upgrade key, security council, pause mechanism, or respected-game configuration can affect how much protection a dispute game actually provides. Review these controls alongside the proof system and contract code.

### Confusing proof of fault with proof of intent

The mechanism can establish that a claim was wrong; it does not need to establish that the proposer acted maliciously. A software bug, incorrect implementation, or honest mistake can also produce a faulty claim.

## 8. Developer Review Checklist

When evaluating an optimistic rollup's fraud-proof design, ask:

- Is the transaction data required for independent re-execution available, and where is it published?
- Can an independent user or node initiate a challenge, and what tools or permissions are required?
- What exactly does the output/state commitment represent?
- How are disputes narrowed and which VM or verifier checks the final step?
- What are the challenge deadlines, game clocks, bond rules, and timeout outcomes?
- Can a faulty claim be finalized or used for a withdrawal before the dispute process finishes?
- Which admin, upgrade, pause, or game-selection powers can change the security assumptions?

Answering these questions is more informative than relying on a marketing label such as “fraud proofs enabled.”

## 9. Key Takeaways

- Fraud proofs—often called **fault proofs**—let a protocol challenge an invalid optimistic-rollup state claim.
- Independent re-execution, access to the required data, and a valid challenge path are essential.
- Interactive games can narrow a large execution disagreement to a small step that an on-chain verifier can check.
- A claim or root is not automatically correct just because it was published; correctness follows from the rollup's dispute and verification rules.
- Fraud-proof security is distinct from data availability, censorship resistance, and bridge finalization.
- Always evaluate the concrete rollup specification, dispute contracts, verifier, deadlines, bonds, and upgrade controls.

## 10. References

- [Ethereum.org — Optimistic Rollups](https://ethereum.org/developers/docs/scaling/optimistic-rollups/) — overview of optimistic rollups, challenge periods, data availability, settlement, and interactive fraud proving.
- [OP Stack Specification — Fault Proof](https://specs.optimism.io/fault-proof/) — architecture of the program, VM, dispute game, and preimage oracle.
- [OP Stack Specification — Fault Dispute Game](https://specs.optimism.io/fault-proof/stage-one/fault-dispute-game.html) — claim semantics, game moves, trace bisection, clocks, and resolution.
- [OP Stack Specification — Dispute Game Interface](https://specs.optimism.io/fault-proof/stage-one/dispute-game-interface.html) — dispute-game contracts and resolution statuses.

[Back to Stage 07 index](README.md) · [Full roadmap](../../roadmap/roadmap.md)
