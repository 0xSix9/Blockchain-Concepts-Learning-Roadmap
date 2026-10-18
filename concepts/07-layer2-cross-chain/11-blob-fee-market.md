# Blob Fee Market: Pricing Ethereum's Rollup Data Space

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

![Blob fee market lifecycle: rollups publish blob-carrying transactions, blob gas demand feeds the excess blob gas accumulator, and the resulting blob base fee changes with capacity pressure. Execution gas remains a separate charge, while EIP-7918 adds a reserve-price relationship to avoid an unresponsive blob market.](../../assets/diagrams/11-blob-fee-market.svg)

## 1. Overview

The **blob fee market** is Ethereum's separate pricing mechanism for the data blobs introduced by EIP-4844. Blobs give rollups a more cost-effective way to publish transaction data to Ethereum without placing all of that data directly in ordinary EVM calldata. Instead of paying only the normal execution-gas market price for transaction bytes, a blob-carrying transaction pays a blob fee computed from the amount of blob gas it consumes and the current blob base fee.

The market exists because rollups are major consumers of Ethereum data availability. They execute most transactions on L2, then publish compressed data that independent participants need to reconstruct and verify the L2 chain. Pricing this data resource separately allows the protocol to adjust blob-space demand independently from ordinary transaction execution, while preserving capacity limits for the network.

However, the two fee dimensions are related in practice. A blob-carrying transaction still consumes L1 execution gas to validate and include the transaction, and Ethereum's Fusaka upgrade activated EIP-7918, which places a reserve-price constraint on the blob fee relative to execution costs. This prevents the nominal blob price from collapsing to the minimum simply because execution costs dominate the total price paid by blob users. The blob market is therefore a separate market with a protocol-defined interaction with execution costs—not a guarantee that blob data will always be nearly free.

## 2. What is a blob?

A **blob** is a fixed-size data object attached to a type-3 blob-carrying transaction. Blob data is made available through Ethereum's consensus-layer data-availability machinery, but it is not directly accessible to EVM contracts as ordinary call data. The EVM transaction contains versioned hashes referring to blob commitments rather than the full blob payload itself.

This design serves rollups well: they can publish large data payloads in a specialized data channel, while contracts can still use commitments and proofs where necessary. Blobs are intended for temporary data availability, not permanent execution-layer storage. The protocol defines a retention window of roughly 18 days under current specifications; applications that need long-term historical blob data must rely on archival services or preserve it independently.

A blob-carrying transaction still includes ordinary transaction fields and consumes normal execution gas. The payload's blob capacity is priced in blob gas, which is tracked separately. Smart contracts do not simply read blob bytes using ordinary calldata operations.

## 3. Execution gas and blob gas are different

Ethereum now has separate resource dimensions relevant to a blob-carrying transaction:

- **Execution gas** pays for normal EVM execution and transaction-processing work on the execution layer. The ordinary execution base fee and any priority fee apply to the transaction's execution-gas usage.
- **Blob gas** measures the blob-space resource consumed by the transaction. Each blob corresponds to a protocol-defined amount of blob gas, and the blob fee is calculated from that amount multiplied by the blob base fee.
- **Other operational costs** can include L2 sequencing fees, compression overhead, proving costs, and the batcher's infrastructure. These are not all protocol-level blob fees, even if they affect the end user's L2 transaction fee.

A useful conceptual formula is:

**Total L1 publication cost ≈ execution-gas charge + blob-gas charge + transaction-related overhead**

This is a model, not a complete transaction accounting formula. The actual charge depends on execution gas used, the execution base fee, priority fee, number of blobs, blob base fee, and the transaction's other parameters. Some of these costs are protocol-level charges and others are rollup operator costs passed on to L2 users.

The key developer lesson is that **low blob base fee does not imply zero L1 transaction cost**. A transaction carrying blobs still competes for execution resources and must pass the required validation checks. Likewise, normal L1 gas can rise even while the blob fee remains low, and blob congestion can raise the blob charge even if ordinary execution gas is comparatively cheap.

## 4. How the blob base fee adjusts

EIP-4844 defines a fee-adjustment mechanism inspired by EIP-1559. It uses two key header fields:

- **blobGasUsed** records the amount of blob gas consumed by a block.
- **excessBlobGas** tracks accumulated usage above the blob gas target, with the protocol's update rules controlling how this accumulator changes from block to block.

The blob base fee is derived from excess blob gas with an exponential-style function. The exact calculation is defined in the Ethereum specifications; conceptually, the accumulator translates persistent demand pressure into a price signal.

### When usage is below target

If total blob usage stays below the target over successive blocks, excess blob gas trends downward, bounded at zero. The computed blob base fee then trends toward its protocol minimum. This gives rollups cheaper data publication when capacity is available.

### When usage is above target

If blocks consistently use more blob gas than the target, excess blob gas accumulates. A higher excess value raises the computed blob base fee, making additional blob consumption more expensive and giving rollups an incentive to compress data more aggressively, delay non-urgent publication, or reconsider their data-publishing configuration.

### Target versus maximum

The **target** is the utilization level the market tries to regulate around over time. The **maximum** is the block-level ceiling; a block can contain more blobs than the target, but not beyond its active protocol limit. A single block above target does not necessarily mean congestion will persist. Sustained usage relative to the target is the more useful signal.

Do not hard-code old blob limits into an application or dashboard. After the Pectra upgrade and the Fusaka BPO changes, Ethereum's blob capacity is managed by a schedule of protocol parameters. The active target, maximum, and fee-update parameters depend on the network's current fork configuration.

## 5. The importance of excess blob gas

The excess accumulator makes the fee depend on recent historical demand instead of only the current block's blob count. If every new block used an instantaneous auction that ignored earlier usage, prices could be more jumpy and less consistent with sustained congestion.

A simplified way to think about the accumulator is:

- If previous usage was above target, excess blob gas carries that pressure forward.
- If usage stays below target, excess blob gas can decline toward zero.
- The blob base fee is computed from the accumulator, using the protocol's configured update fraction.
- The current block's blob transaction pays the fee defined by the applicable protocol state, rather than picking an arbitrary fee per blob.

This is not exactly the same transaction-by-transaction auction as ordinary EIP-1559 execution gas. The model is designed around block-level blob usage and an evolving base fee. A transaction may set a maximum blob fee it is willing to pay; if the fee required by the network exceeds that limit, the transaction cannot be included under those conditions.

The protocol's constants and update fractions have changed or may change across upgrades. In particular, blob target/max limits and the base-fee update fraction are managed through the blob schedule. Engineering tools should use the correct active fork parameters rather than assume the original EIP-4844 configuration forever.

## 6. Fusaka, PeerDAS, and EIP-7918

The Fusaka upgrade, activated on Ethereum Mainnet on December 3, 2025, changed how Ethereum scales blob availability and pricing. Two features are especially relevant to understanding the market.

### Blob Parameter Only (BPO) forks

EIP-7892 provides a way to adjust blob-related parameters through specialized upgrades, without bundling every increase into a larger named hard fork. This enables blob capacity to increase gradually as node and network performance is tested. The target and maximum values are configuration values that can change over time.

The Ethereum Foundation's January 2026 update reported that the first two BPO changes had raised mainnet capacity to a target of 14 blobs per block and a maximum of 21. Treat these values as a documented configuration snapshot, not immutable constants for every Ethereum network or future fork. For applications, explorers, and fee estimators, the authoritative source is the active schedule for the network being queried.

### PeerDAS

EIP-7594, part of Fusaka, introduced Peer Data Availability Sampling (PeerDAS). Rather than requiring every node to download and store every full blob, nodes custody subsets of data and sample columns across the network. Erasure coding and sampling help nodes establish that the blob data is available without forcing each regular node to handle every byte.

This improves the feasibility of raising blob throughput while keeping hardware and bandwidth requirements more manageable. PeerDAS supports scalability of the blob space, but it does not make blob data permanent. Availability is guaranteed under the protocol's data-availability model and retention rules.

### EIP-7918: blob base fee bounded by execution cost

Before EIP-7918, the blob fee could become a weak price signal when execution costs dominated the total cost of posting blobs. The ordinary blob fee adjustment might continue lowering the nominal blob base fee toward its minimum even when blob transactions remained expensive because of execution gas and blob-verification work. That could allow demand for blobs to remain high without the blob fee reacting promptly.

EIP-7918 introduces a reserve-price relationship tied to execution costs. When the nominal blob base fee would be too low relative to the execution base fee and the defined per-blob base cost, the update rule stops reducing excess blob gas as it otherwise would. This lets excess pressure build and the blob fee respond to usage, while keeping the defined per-block usage constraints.

Conceptually, this has two purposes:

1. **Keep the blob fee market responsive.** If execution gas is the dominant portion of the total price, the market should not lose its signal and remain stuck at the minimum blob base fee during high demand.
2. **Price some of the computational burden imposed by blobs.** Nodes must validate blob-related commitments and proofs. The reserve-price relationship means blob users contribute a defined minimum share toward the execution resources the transaction imposes, without replacing the blob market's independent data-space price.

The precise branch and formula are protocol-defined; use EIP-7918 and the active execution specifications for implementation. This article explains the intended economic behavior rather than reproducing a version-specific client algorithm.

## 7. Why the blob fee market matters to rollups

Blob pricing directly influences the cost of publishing L2 transaction data to Ethereum. If blob space is plentiful and the blob base fee is low, a rollup can post batches more cheaply. If demand reaches and sustains usage above target, the increasing price creates incentives to optimize data representation or change batch timing.

Rollups can respond to blob congestion in several ways:

- **Compress transaction data more effectively**, reducing the bytes required to represent many L2 transactions.
- **Batch more transactions per publication**, amortizing fixed L1 transaction overhead across a larger group, although waiting for a larger batch can increase publication latency.
- **Adjust publication policy**, provided they remain within protocol-specific deadlines and sequencing-window requirements.
- **Use available capacity strategically**, distinguishing urgent data from workloads that can tolerate more delay.
- **Improve data encoding**, avoiding redundant fields where the rollup protocol permits compression without changing deterministic derivation.

The optimization space has constraints. Rollups cannot safely omit data required for independent reconstruction, proofs, or dispute resolution. A lower fee obtained by weakening the rollup's data-availability assumptions may change the security model and is not equivalent to an efficiency improvement within an Ethereum rollup.

## 8. What blob fees do not tell you

Blob base fee is a useful metric, but it is not the whole rollup fee or security picture.

- **It is not the L1 execution base fee.** A transaction can have low blob fees but high execution fees, or the opposite.
- **It is not an L2 transaction's final user fee.** Rollup operators may charge execution, proving, sequencing, or other service costs in addition to data publication.
- **It is not a direct measure of DA safety.** Data availability follows protocol rules and the network's actual operation; a low price does not mean the data is less secure, and a high price does not itself prove availability.
- **It is not a direct estimate of permanently stored bytes.** Blobs have a limited retention period in normal client operation; long-term archival is a separate service.
- **It is not fixed across forks.** Active blob capacity and fee-update parameters can change through scheduled upgrades.
- **It is not a guarantee of cheaper L2 fees under every condition.** L2 execution demand, sequencing fees, batcher behavior, and rollup competition also affect what users pay.

## 9. Developer and operator checklist

When analyzing a rollup's blob costs or building a fee estimator, check:

- Does the rollup publish its data in blobs, calldata, or a mixture that changes by batch or protocol version?
- What is the active network's blob schedule, including target, maximum, and fee-update fraction?
- Is the transaction type blob-carrying, and how many blobs does it reference?
- What is the current blob base fee, derived from the correct parent/block context?
- What execution gas does the transaction consume, and what execution base fee and priority fee apply?
- Does the estimate include batcher transaction overhead, proof verification, and other execution-layer costs?
- Does the rollup's compression and batching policy preserve all data required for reconstruction and the protocol's challenge mechanism?
- Are dashboards separating blob gas usage, excess blob gas, blob base fee, execution base fee, and end-user L2 fees?
- Have upgrades such as Fusaka, BPO parameter changes, and EIP-7918 been accounted for in the implementation?
- Is the service correctly handling transactions whose maximum blob fee is below the active blob base fee?
- Does the rollup have a fallback publication policy for congestion, and does it remain within the protocol's data-publication deadlines?

## 10. Key takeaways

- EIP-4844 introduced blobs and a separate blob gas market for rollup data publication.
- The blob base fee responds to excess blob gas relative to a target; the target and maximum are different parameters.
- A blob-carrying transaction still pays execution gas in addition to its blob fee.
- Fusaka introduced PeerDAS and BPO scaling, and activated EIP-7918's execution-cost-bounded blob base-fee mechanism.
- Blob parameters are protocol configuration, not constants that applications should hard-code across all forks and networks.
- Blob fees influence L2 data costs, but are only one component of users' total transaction fees.
- Blobs are a data-availability resource with limited retention, not permanent EVM storage.

## References

1. [EIP-4844 — Shard Blob Transactions](https://eips.ethereum.org/EIPS/eip-4844)
2. [EIP-7691 — Blob Throughput Increase](https://eips.ethereum.org/EIPS/eip-7691)
3. [EIP-7840 — Blob Schedule for Network Upgrades](https://eips.ethereum.org/EIPS/eip-7840)
4. [EIP-7892 — Blob Parameter Only Hardforks](https://eips.ethereum.org/EIPS/eip-7892)
5. [EIP-7918 — Blob Base Fee Bounded by Execution Cost](https://eips.ethereum.org/EIPS/eip-7918)
6. [Ethereum.org — Fusaka Upgrade](https://ethereum.org/roadmap/fusaka/)
7. [Ethereum.org — PeerDAS](https://ethereum.org/roadmap/fusaka/peerdas/)
8. [Ethereum Foundation — Fusaka Mainnet Announcement](https://blog.ethereum.org/2025/11/06/fusaka-mainnet-announcement)
9. [Ethereum Foundation — Checkpoint #8, January 2026](https://blog.ethereum.org/2026/01/20/checkpoint-8)
10. [Ethereum.org — Optimistic Rollups and Fees](https://ethereum.org/developers/docs/scaling/optimistic-rollups/)
