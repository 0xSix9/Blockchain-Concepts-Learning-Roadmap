# State Commitments: Roots That Represent Rollup State

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

## 1. Overview

A state commitment is a cryptographic value—often a root hash—that commits to a structured representation of a rollup's state at a particular point. If the state is represented by a Merkle-style tree, the root changes when relevant state changes and can be used in proofs about individual values. Rollup contracts commonly store or accept such commitments as part of their settlement logic.

![State Commitments: Roots That Represent Rollup State](../../assets/diagrams/05-state-commitments.svg)

## 2. Why It Matters to a Developer

State commitments let L1 contracts refer compactly to a large L2 state without storing every account and storage slot on Ethereum. They are used in withdrawal proofs, validity-proof public inputs, and optimistic dispute systems. Understanding exactly what a root commits to prevents developers from confusing a hash with proof that the state is correct.

## 3. How It Works

1. A rollup defines a state representation and deterministic transition rules.
2. Executing transactions produces a new state; the implementation derives a commitment to that state, commonly a root hash.
3. The rollup submits the root with a batch, assertion, or proof-related record.
4. A verifier or dispute protocol determines whether the transition to that root is acceptable.
5. Later operations—such as a withdrawal proof—use the accepted commitment as an anchor for verifying a specific claim.

A root is binding under the relevant cryptographic assumptions, but the root itself does not tell a verifier whether the committed state was computed correctly. Correctness comes from the rollup's proof or dispute mechanism.

## 4. Example or Experiment

Suppose the accepted root is `R`. A withdrawal proof shows that a specific withdrawal message is included in the message-passer state associated with `R`. The bridge must still verify that `R` is an accepted output for the correct chain and that the withdrawal has not already been finalized. Inclusion under an arbitrary root is not enough.

## 5. Common Mistakes and Security Considerations

Verify which data is committed: account state, withdrawal messages, batch transactions, or an output structure may use different roots. Domain-separate commitments and bind them to chain IDs, block numbers, and protocol versions where required. Root correctness can also be undermined by a faulty transition function, weak data availability, untrusted upgrades, or an incorrect verifier.

## 6. Key Takeaways

- A root compactly commits to a particular data structure or state snapshot.
- A commitment is not, by itself, a proof that execution was valid or that data is available.
- Withdrawal and proof verification must bind the root to the correct chain, accepted output, and message.

## 7. References

- [Ethereum.org — Optimistic Rollups](https://ethereum.org/developers/docs/scaling/optimistic-rollups/)
- [Ethereum.org — Zero-Knowledge Rollups](https://ethereum.org/developers/docs/scaling/zk-rollups/)
- [Optimism Specs — Withdrawals](https://specs.optimism.io/protocol/withdrawals.html)
