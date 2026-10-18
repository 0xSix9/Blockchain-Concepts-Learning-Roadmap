# Batch Submission: Publishing Rollup Data to Layer 1

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

## 1. Overview

Batch submission is the process of publishing data or commitments representing many L2 transactions to the settlement layer. Aggregating transactions spreads L1 overhead across many operations and lets a rollup compress repetitive fields. What is submitted matters: a batch may include transaction data, a state commitment, a proof, or several of these, and the exact design varies.

![Batch Submission: Publishing Rollup Data to Layer 1](../../assets/diagrams/04-batch-submission.svg)

## 2. Why It Matters to a Developer

For many Ethereum rollups, L1 data publication is a major variable cost. Developers who understand the batch path can reason about fee allocation, batch latency, reorgs, data availability, and why a cheap L2 execution can still incur publication costs.

## 3. How It Works

1. The sequencer orders L2 transactions and creates blocks or batches.
2. A batcher encodes and often compresses the transaction data using the rollup's format.
3. The batch is submitted to Ethereum, historically often as calldata and, where supported, through blob-carrying transactions.
4. L1 consensus makes the submitted data available according to the relevant data-availability mechanism; the execution layer processes the transaction or blob commitment metadata.
5. Rollup nodes reconstruct the same L2 history from the published data, while proof or fraud-proof logic establishes state-transition correctness.

Data publication and state correctness are related but distinct. Publishing transaction data helps others independently recompute the state; a state root alone is not a replacement for the underlying data.

## 4. Example or Experiment

If a batch contains 1,000 transactions, the rollup can amortize the L1 transaction's fixed overhead over those transactions. But each user's cost still depends on compressed bytes, current calldata/blob pricing, batcher strategy, proof costs, and how the rollup allocates shared costs. A larger batch may reduce average cost while increasing the delay before publication.

## 5. Common Mistakes and Security Considerations

Do not assume every rollup publishes all data on-chain. Validiums and hybrid designs can use off-chain data availability and add different trust assumptions. Also distinguish the lifetime and retrieval rules for blobs from permanent EVM-accessible calldata, and verify what happens during sequencer or batcher downtime.

## 6. Key Takeaways

- Batch submission amortizes overhead and anchors rollup data or commitments to L1.
- Data publication, proof verification, and sequencing are different pipeline stages.
- Fee and finality estimates must use the specific rollup's batching policy and publication medium.

## 7. References

- [Ethereum.org — Data Availability](https://ethereum.org/developers/docs/data-availability/)
- [EIP-4844 — Blob-carrying Transactions](https://eips.ethereum.org/EIPS/eip-4844)
- [Ethereum.org — Optimistic Rollups](https://ethereum.org/developers/docs/scaling/optimistic-rollups/)
