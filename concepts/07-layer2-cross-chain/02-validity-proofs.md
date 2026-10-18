# Validity Proofs: Cryptographically Verifying Rollup Execution

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

## 1. Overview

A **validity proof** is a cryptographic proof that a computation satisfies a precisely defined statement. In a validity rollup, an operator executes a batch of Layer 2 (L2) transactions and produces a proof that the resulting state transition follows the rollup's rules. A verifier on Ethereum Layer 1 (L1) checks that proof and the public inputs bound to it. If verification and the contract's other conditions succeed, the rollup can accept the new state commitment.

The key idea is that Ethereum does not need to re-execute every L2 transaction just to check the proof. Instead, a prover performs the computationally expensive work offchain and produces cryptographic evidence that a verifier can check more efficiently.

![Validity proofs: execute transactions, generate a proof, verify public inputs, and accept the state commitment](../../assets/diagrams/02-validity-proofs.svg)

The phrase **ZK-rollup** is commonly used for validity-proof rollups. However, two properties should be kept separate:

- **Validity:** the proof convinces the verifier that a specified computation or state transition satisfies the rules encoded by the proof system.
- **Zero knowledge:** the proof reveals no information about the witness beyond what is implied by the public statement.

A rollup can use validity proofs to verify execution without making user transactions private. Most public rollup activity remains observable through the data or transaction information the protocol publishes.

## 2. Why Validity Proofs Matter to a Developer

Validity proofs move most transaction execution and proof generation away from Ethereum's base execution layer while retaining an onchain verification step. This can make it practical to process many L2 transactions while asking Ethereum to verify a compact proof of the resulting transition.

For developers, validity proofs help explain:

- **State correctness:** why a new state root can be accepted without replaying all L2 transactions inside the L1 verifier.
- **Settlement:** when a state commitment becomes acceptable under the rollup contract's proof-verification and finalization rules.
- **Proof-system trade-offs:** why proof size, proving time, verification gas, cryptographic assumptions, and hardware requirements vary.
- **The limits of cryptographic guarantees:** why a valid proof cannot, by itself, guarantee data availability, censorship resistance, correct bridge code, or safe upgrade controls.

A validity proof proves only the statement that the implemented circuit or virtual machine actually checks. It does not make bugs in that statement disappear.

## 3. The Statement: What Exactly Is Being Proven?

A proof system works with a **statement** and a **witness**.

- **Statement / public inputs:** information visible to the verifier and cryptographically bound to the proof. For a rollup, this may include the previous state root, proposed new state root, batch or data commitment, and other protocol-specific values.
- **Witness:** the underlying information needed to construct the proof. Depending on the system, it can include transaction data, intermediate execution states, Merkle paths, signatures, and other data used by the computation.
- **Prover:** software or infrastructure that generates a proof showing the statement satisfies the specified relation.
- **Verifier:** an algorithm, often called by an L1 verifier contract, that checks the proof against the public inputs and verification parameters.

A simplified rollup statement might be described as:

`ApplyBatch(oldRoot, batchData) = newRoot`

The proof should establish that the transition from `oldRoot` to `newRoot` follows the rollup's specified rules for `batchData`. The actual public inputs and encoding are protocol-specific; this expression is conceptual pseudocode, not a universal verifier API.

Cryptographic commitments help bind the proof to the intended inputs. A verifier must not accept a proof for one batch while accidentally applying it to a different state root, chain, or commitment. The exact statement, input layout, and domain separation need to be read from the target rollup's specification.

## 4. How a Validity Rollup Works

A typical validity-rollup pipeline has these steps:

1. **Collect and order transactions.** Users send transactions through the rollup's supported submission path. The operator or sequencer applies protocol checks and determines an order.
2. **Execute on L2.** The execution system processes the transactions from the known pre-state and derives the proposed post-state.
3. **Prepare the proving inputs.** The system encodes the computation into the relation expected by its proving system. It may include state roots, transaction commitments, intermediate state information, and other public or private inputs.
4. **Generate a proof.** A prover produces cryptographic evidence that the statement satisfies the relation. This step may take substantial computation and specialized infrastructure.
5. **Publish data and submit the proof.** For a rollup that uses Ethereum for data availability, the relevant transaction data is published to L1, historically as calldata and also through blob-carrying transactions under EIP-4844. The operator submits the proof and protocol-specific commitments to the appropriate contracts.
6. **Verify and accept the transition.** The L1 verifier checks the proof against its public inputs. The rollup contract applies its remaining checks and updates or accepts the state commitment if every required condition passes.
7. **Settle under the L1 rules.** The accepted update is included in Ethereum's chain. The transaction and resulting state are subject to Ethereum's own consensus and finality process.

Exact batching, proof submission, data publication, and state-update logic varies between rollups. Some systems aggregate many batches into a proof; others use recursive proofs or multiple stages of proof aggregation.

## 5. Worked Example: Alice Sends 10 Tokens to Bob

Assume the accepted L2 state contains Alice's balance and Bob's balance. A transaction transfers 10 units from Alice to Bob.

Before the batch:

- Alice has 50 units.
- Bob has 7 units.
- The state is represented by a prior root, `R0`.

After applying the transaction according to the rollup's rules:

- Alice has 40 units.
- Bob has 17 units.
- The resulting state is represented by a proposed root, `R1`.

The operator cannot make `R1` trustworthy simply by publishing the hash. It must provide a proof whose statement binds the prior root, the relevant transaction or batch commitment, the proposed new root, and every other public input required by that rollup. The proof system's relation checks the transaction conditions and state updates, including the applicable balance, signature, nonce, and state-transition rules.

A simplified verifier call might look like:

`verifyProof(proof, oldRoot, newRoot, batchCommitment)`

This is illustrative pseudocode only. Actual verifier contracts may use different ABI arguments, encodings, verification keys, recursion layers, and state-update logic. A correct implementation must also prevent replay and bind the proof to the correct chain and protocol context where applicable.

If the proof verifies and the rollup contract's other conditions pass, the state commitment can be accepted. If the proof fails, the contract must not accept that proposed transition through the proof-verification path. The contract still needs correct handling of invalid inputs, duplicate submissions, and state-update ordering.

## 6. SNARKs, STARKs, and Proof Engineering

Validity proofs are a function, not one particular cryptographic construction. Rollups can use different proof systems, each with its own security assumptions and engineering trade-offs.

### SNARKs

A **succinct non-interactive argument of knowledge (SNARK)** can produce a compact proof that is efficiently verified. Some SNARK systems rely on a trusted setup ceremony to create public parameters. Security depends on the ceremony's assumptions, the cryptographic construction, and correct implementation. Not every SNARK has the same setup model.

### STARKs

A **scalable transparent argument of knowledge (STARK)** is designed around transparent setup assumptions and uses cryptographic techniques that differ from many SNARK constructions. STARK proof sizes and proving costs can differ significantly from other systems; “transparent” does not mean the system has no cryptographic assumptions or implementation risk.

### What a developer should compare

When evaluating a rollup or a proof system, consider:

- **Proving time and hardware:** how expensive it is to generate proofs, and whether proof production depends on a small set of operators.
- **Verification cost:** how much L1 gas the verifier and associated state-update logic consume.
- **Proof size and aggregation:** how efficiently the system can verify many transitions or recursively aggregate proofs.
- **Security assumptions:** the underlying cryptography, setup model where applicable, soundness assumptions, and implementation maturity.
- **Circuit or VM coverage:** whether the verified relation accurately implements the intended transaction and state-transition rules.

No single label—SNARK, STARK, or ZK—answers all of these questions.

## 7. Validity Is Not the Same as Data Availability

**Validity proof and data availability solve separate problems.**

A proof can establish that a state transition follows the encoded rules, but users also need access to the data required to reconstruct the state, inspect their balances, submit new transactions, or produce withdrawal proofs.

- A **validity rollup** that publishes its transaction data to Ethereum lets independent participants use the published data to reconstruct the rollup's state, subject to the protocol's data format and availability assumptions.
- A **validium** uses validity proofs but keeps the relevant data outside Ethereum's L1 data-availability layer, introducing a separate data-availability assumption.
- A **volition** design may offer more than one data-availability mode, with the choice changing the assumptions for the relevant state or transactions.

This distinction matters when assessing the security model. A validity proof does not force an external data-availability committee to reveal withheld data. Nor does data being published onchain, by itself, prove that the transition was computed correctly.

## 8. Validity Proofs Versus Fraud Proofs

Both proof-based and optimistic systems aim to prevent invalid state transitions from being accepted, but they use different processes.

| Question | Validity-proof model | Optimistic / fraud-proof model |
|---|---|---|
| Default acceptance | The state update must pass proof verification and the contract's rules. | A claim may be provisional under the protocol's rules while a challenge window remains open. |
| How correctness is checked | A verifier checks a cryptographic proof of a specified computation. | A challenger can dispute a claim using a fraud/fault-proof mechanism. |
| Main offchain work | Execution and proof generation. | Execution, independent monitoring, and dispute preparation when needed. |
| Key assumptions | Sound proof system, correct circuit/VM, verifier, input binding, and relevant data availability. | Available data, capable timely challengers, correct dispute game/verifier, and the challenge protocol's assumptions. |
| Privacy | Not implied by validity verification; depends on the proof's properties and what data is published. | Not implied by optimistic execution either; depends on the protocol and published data. |

Neither model removes the need for careful bridge design, censorship-resistance mechanisms, protocol upgrades, or data-availability analysis. The exact security model must be evaluated at the level of the deployed contracts and current specification.

## 9. Common Mistakes and Security Considerations

### Assuming every validity proof provides privacy

“Zero knowledge” is a property of a proof system, while validity is about proving a computation satisfies its rules. A public rollup can publish transaction data and still use a succinct validity proof. Check exactly what information is hidden, what is public, and what the protocol reveals through commitments or events.

### Treating the proof as more general than its circuit or VM

The verifier checks a specific mathematical relation. If the circuit accidentally omits a transaction rule or models EVM behavior incorrectly, a proof can be valid relative to the faulty circuit yet fail to establish the intended protocol behavior. Correct specification, circuit audits, tests, and reproducible builds matter.

### Accepting proofs without binding all public inputs

State roots, batch commitments, chain context, and protocol versions must be correctly bound according to the design. A mismatch in input order, encoding, or domain binding can undermine the connection between the proof and the state update it is supposed to authorize.

### Ignoring setup and implementation assumptions

Some proving systems have setup assumptions; all systems depend on sound cryptographic construction and correct implementations. Review verifier code, parameters, circuit or VM implementation, libraries, and upgrade pathways instead of assuming a proof is infallible.

### Equating proof verification with complete transaction finality

A proof may be generated offchain well before it is submitted. Verification must be accepted by the relevant L1 contract, and the accepted update remains subject to the L1 transaction's inclusion and consensus-finality rules. Bridge withdrawals can also have additional message-proof, replay-protection, and finalization steps.

### Ignoring the operator and data-availability path

A sound verifier does not automatically prevent the sequencer from censoring transactions, guarantee every user can obtain required data, or eliminate upgrade-admin risk. Review the full end-to-end rollup and bridge architecture.

## 10. Developer Review Checklist

When studying a validity-proof rollup, answer these questions:

- What exact statement does the proof establish, and what are its public inputs and witness?
- Which circuit, VM, or state-transition specification is being proven?
- How is the proof bound to the old root, new root, batch, chain, and protocol version?
- What verifier contract and proof-system assumptions are used?
- How are transaction data and other inputs made available so independent users can reconstruct state?
- What happens if proof generation stops, the verifier rejects a proof, or an operator censors transactions?
- Who can upgrade or pause the verifier, settlement contract, bridge, or proof configuration?
- What exactly must happen before an L2 state update or withdrawal is considered finalized?

These questions turn a general claim about “ZK security” into a concrete engineering and threat-model review.

## 11. Key Takeaways

- A validity proof establishes that a specified state transition satisfies the rules encoded by a particular proof system.
- A verifier can check cryptographic evidence without re-executing the entire L2 batch on Ethereum.
- The proof must be tied to the correct public inputs, state commitments, and chain/protocol context.
- SNARKs and STARKs have different engineering and security trade-offs; neither name guarantees a bug-free implementation.
- Validity and zero knowledge are distinct properties: validity proofs do not automatically provide transaction privacy.
- Validity proofs do not replace data availability, censorship resistance, sound bridge contracts, or safe upgrade controls.
- L1 proof acceptance, L1 consensus finality, and bridge withdrawal finalization are distinct milestones.

## 12. References

- [Ethereum.org — Zero-Knowledge Rollups](https://ethereum.org/developers/docs/scaling/zk-rollups/) — validity-proof workflow, state commitments, data availability, and settlement.
- [Ethereum.org — Zero-Knowledge Proofs](https://ethereum.org/zero-knowledge-proofs) — definitions and properties of SNARKs, STARKs, and zero knowledge.
- [Ethereum.org — Data Availability](https://ethereum.org/developers/docs/data-availability/) — why transaction data must be available for independent verification and state reconstruction.
- [ZKsync Protocol — Block State Transition Circuit](https://github.com/matter-labs/zksync/blob/master/docs/protocol.md) — an implementation-oriented example of public inputs, transaction data, state roots, and validity constraints.
- [Starknet Docs — SNOS](https://docs.starknet.io/architecture-and-concepts/network-architecture/os/) — an example of defining block validity as a provable computation.

[Back to Stage 07 index](README.md) · [Full roadmap](../../roadmap/roadmap.md)
