# Sequencers: Ordering Transactions on a Rollup

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

## 1. Overview

A sequencer is an operator or coordinated system that orders user transactions and typically proposes L2 blocks or batches. It may provide quick acknowledgements or preconfirmations before the corresponding data and commitments reach Ethereum. Sequencing is distinct from execution: execution applies transactions to a state, while sequencing determines the order in which transactions are processed.

![Sequencers: Ordering Transactions on a Rollup](../../assets/diagrams/03-sequencers.svg)

## 2. Why It Matters to a Developer

Sequencer design influences latency, fees, censorship resistance, transaction ordering, MEV, liveness, and the ability to recover if the operator stops working. A rollup can use Ethereum for settlement while still relying on a centralized sequencer for the normal path.

## 3. How It Works

1. A wallet or application submits a signed transaction to the rollup's submission endpoint.
2. The sequencer validates basic transaction conditions, chooses an order, and may issue a soft confirmation.
3. L2 execution clients process the ordered transactions and calculate the resulting state.
4. The sequencer or batch poster publishes batch data and/or commitments to L1 according to the rollup design.
5. The settlement contract applies the relevant proof or challenge rules before an output becomes final under that system's model.

A sequencer's soft confirmation is not necessarily equivalent to L1 inclusion, proof acceptance, or finality. Those are separate states.

## 4. Example or Experiment

Imagine two users submit swaps close together. The sequencer's order can change the price each receives, so ordering policy matters even when both transactions are individually valid. A dApp that displays 'confirmed' should document whether that means sequencer acknowledgement, an L2 block, L1 batch inclusion, or protocol-level finality.

## 5. Common Mistakes and Security Considerations

A single operator can censor, reorder, or temporarily halt transactions and may extract MEV. Its outage can affect liveness even if users eventually have an L1 forced-inclusion route. A decentralized sequencer set introduces its own assumptions—quorums, network delay, leader selection, slashing, and coordination—and is not automatically secure merely because it has multiple members.

## 6. Key Takeaways

- Sequencing selects transaction order; execution computes state changes.
- Soft confirmations and settlement finality are not interchangeable.
- Evaluate fallback paths, censorship resistance, ordering policy, MEV, decentralization, and operator privileges.

## 7. References

- [Ethereum.org — Optimistic Rollups](https://ethereum.org/developers/docs/scaling/optimistic-rollups/)
- [Ethereum.org — Zero-Knowledge Rollups](https://ethereum.org/developers/docs/scaling/zk-rollups/)
- [Based Sequencing](https://www.basedsequencing.org/)
