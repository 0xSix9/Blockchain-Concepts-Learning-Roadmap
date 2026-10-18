# PeerDAS: Ethereum's Peer Data Availability Sampling

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

![PeerDAS architecture: blobs are extended with erasure coding and split into cells and columns; consensus nodes receive selected columns through gossip and peer requests, sample columns for availability, and custody a deterministic subset for serving other nodes.](../../assets/diagrams/13-peerdas.svg)

## 1. Overview

**PeerDAS (Peer Data Availability Sampling)** is Ethereum's specific peer-to-peer protocol for checking blob data availability without requiring every node to download every blob in full. It applies the general idea of Data Availability Sampling (DAS) to the blobs introduced by EIP-4844. PeerDAS extends blob data with erasure coding, subdivides the extended representation into authenticated cells, organizes the cells into columns, and distributes those columns among peers. Nodes sample columns to check availability and keep custody of an assigned subset so other nodes can obtain the data they need.

PeerDAS was activated on Ethereum Mainnet as part of the Fusaka upgrade on December 3, 2025. It is a key component of Ethereum's plan to raise blob capacity for rollups while keeping the bandwidth burden on an individual node more manageable. Instead of making every full node retrieve all blob bytes for every block, the protocol spreads data across the peer network and makes randomized checks possible.

PeerDAS is not simply a new fee market, a new blob type, or a validity-proof system. It changes the way blob data is distributed, sampled, and held by consensus-layer peers. Blob transactions and their commitments remain part of the wider EIP-4844 design; PeerDAS provides a scalable availability mechanism around that data.

## 2. The problem PeerDAS solves

Before PeerDAS, a node that wanted to verify blob availability had to download the full blob data. That is a straightforward local check: the node can only confirm the full payload was available to it after actually receiving it. However, if blob capacity increases substantially, requiring every node to download every byte makes network bandwidth scale directly with total blob throughput.

This creates a scalability trade-off. Higher rollup data throughput can reduce L2 publication costs, but it can also increase bandwidth, processing, and storage pressure on nodes. If ordinary nodes need increasingly powerful network connections just to keep up, running independent nodes may become less accessible, potentially concentrating participation.

PeerDAS seeks to break the one-node-downloads-every-byte relationship. Each node receives and checks a subset of the extended data. The network's aggregate distribution and the protocol's sampling rules create a probabilistic check for withholding, while redundancy means the original data can be recovered from a sufficient subset of the extension.

The goal is not to eliminate bandwidth, erasure coding, or data storage. It distributes the work across a diverse peer network and changes what each node must download to establish an availability check.

## 3. The data model: blobs, cells, and columns

PeerDAS's central abstraction is a **data column**. To understand columns, start with a blob and follow the way its data is represented.

### Blobs and KZG commitments

A blob is a fixed-size data payload associated with an EIP-4844 blob-carrying transaction. The transaction references versioned hashes derived from KZG commitments; the full blob payload is not exposed to contracts as normal EVM calldata. KZG commitments make it possible to authenticate data against a committed blob without requiring each verifier to trust the peer supplying it.

### Erasure-extended blobs

PeerDAS applies a one-dimensional erasure-coding extension to the blob data. In simplified terms, data is treated as evaluations of a polynomial and extended with additional evaluations. The result has redundancy: if enough valid parts remain available, the original blob can be reconstructed even if other parts are missing.

The extended representation is organized into rows, with each row corresponding to one blob and its extension. This is the foundation for cells and columns.

### Cells

An extended blob row is divided into **cells**, the smallest units used by the PeerDAS design for individual KZG authentication. Each cell has data and an associated proof that can be verified against the blob's KZG commitment at the appropriate position.

Cells matter because they are small, authenticated chunks of the extended blob. A node can retrieve selected cells instead of downloading and checking an entire blob first.

### Columns

A **column** combines cells at the same column index across the blob rows in a block. Think of the extended data as a matrix: each blob contributes a row, and a column takes a vertical slice through the rows at one index. Columns are associated with their own gossip subnets and form the units used for PeerDAS sampling and custody.

A column is not a separate original blob. It is a cross-section of the extended data across blobs in a block. Sampling columns lets nodes check different parts of the block's blob data without every node obtaining the whole matrix immediately.

The current Fulu consensus specifications define 128 columns in the extended matrix. This is a protocol parameter, not a value application developers should assume can never change. Always consult the active consensus specification and fork configuration when writing client software.

## 4. How PeerDAS works

The following sequence captures the high-level operation. Details such as validation order, message formats, and parameter choices are defined in the consensus specifications.

1. **Construct blob data.** Blob-carrying transactions provide blob payloads and commitments. A block producer or client assembles the blob data associated with the block.
2. **Extend each blob.** Erasure coding adds redundancy to the original data so the extended representation can recover the original when enough valid pieces are available.
3. **Build cells and columns.** The extended blobs are divided into cells. Cells with matching column indices across the blobs are grouped into columns.
4. **Authenticate the pieces.** Cell KZG proofs enable a node to verify that a received cell is consistent with the relevant blob commitment and index.
5. **Publish data column sidecars.** The consensus client distributes column data with the context and proofs needed to validate it. A DataColumnSidecar includes the column index and column, associated KZG commitments and proofs, signed block header, and an inclusion proof for the commitments.
6. **Gossip to the relevant subnets.** Peers subscribe to data-column subnets according to their custody assignments and share relevant columns.
7. **Request samples.** Nodes choose columns to sample for a slot and retrieve them from peers. If gossip did not deliver a required column, the node can request it through the peer request/response protocol or try another appropriate peer.
8. **Validate samples.** The client checks the structure, inclusion proof, and KZG proofs before relying on a received sidecar.
9. **Check availability and custody.** A node's sampling result supports its local availability check. Separately, it retains its deterministic custody subset and serves those columns to other peers during the relevant retention period.

The important engineering distinction is that **sampling and custody are related but not identical tasks**. Sampling is the local check a node performs to gain confidence that blob data is available. Custody is the subset of columns that a node is responsible for keeping and serving to support other nodes and the network's distribution.

## 5. Sampling and reconstruction

PeerDAS's sampling strategy operates over columns, not over complete blobs. Each node selects a set of columns to request and validates the returned data. Under the current Fulu specification, SAMPLES_PER_SLOT is defined as 8, with the effective sampling size also depending on the node's advertised custody-group count. Parameter values belong to the active protocol configuration and should be checked against the version a client implements.

The redundancy in the extended matrix allows reconstruction from enough columns. The specification states that the complete data matrix can be reconstructed once at least 50% of all columns have been acquired. A node that has less than the reconstruction threshold can request additional columns from peers if it needs to reconstruct more data. Sampling alone does not mean each node has downloaded enough columns to reconstruct every blob in every block.

Why can fewer columns be useful? The local sampling check and full reconstruction have different goals. Sampling asks whether randomly selected pieces are obtainable, which helps detect withholding. Reconstruction asks for enough pieces to recover the original data. A node that is only sampling a subset can still contribute to the distributed availability check without downloading the entire matrix.

This difference avoids a common misconception: **a successful sample is not the same as reconstructing a complete blob locally**. Sampling provides a probabilistic signal under the protocol's security assumptions. Reconstruction requires a sufficient amount of compatible data.

## 6. Custody groups and deterministic responsibility

PeerDAS needs more than random download requests. If nodes sampled pieces but no one had a clear responsibility to retain and serve them, a node could repeatedly request a column that no available peer possessed. Custody gives the network a structured distribution of responsibility.

The consensus specification groups columns into **custody groups**. A node's custody groups are selected deterministically from public node identity information and the custody size it advertises. This lets peers discover which nodes are likely to serve which groups without a central allocator.

The current mainnet Fulu configuration lists 128 possible custody groups, a base `CUSTODY_REQUIREMENT` of 4 groups for an honest non-validator node, and a `VALIDATOR_CUSTODY_REQUIREMENT` of 8 groups for a node with one or more validators attached. The required custody for validator-attached nodes can increase with the total effective balance of those validators, up to the protocol maximum. These are active configuration values, not universal guarantees for future forks or other networks. Nodes can choose to custody more than the minimum; that increases their data-serving contribution and may improve practical availability and retrieval performance.

A node advertising additional custody is responsible for storing and serving the corresponding columns. Its selection of custody groups extends according to the specification's deterministic function, and its peer-discovery record advertises the custody amount. Validators have a custody requirement that scales with validator count, reflecting their role in providing a more robust backbone for the network.

Custody is therefore not merely an optional cache. It is a protocol-defined responsibility connected to peer discovery, gossip subnet participation, and the ability to serve column data.

## 7. Gossip, peer discovery, and request/response

PeerDAS uses a combination of gossip and direct peer requests:

- **Gossip distribution:** data column sidecars are published through column-specific gossip topics. Nodes subscribe to the subnets associated with their custody groups and propagate relevant columns.
- **Peer discovery:** nodes use discovery metadata, including advertised custody-group information, to find peers likely to have the columns they need.
- **Request/response fallback:** if required columns are missing from gossip, clients can request sidecars by block root and column index from peers that may have them.
- **Diversity and retry behavior:** maintaining a diverse peer set and trying alternative peers helps reduce dependence on one failing or malicious peer.

This network layer matters to the security and liveness of sampling. A valid column that cannot be retrieved due to temporary peer failures is different from a cryptographic proof that is invalid. Clients need to distinguish network failures, missing data, malformed sidecars, and failed authentication checks so they can respond correctly.

PeerDAS cannot make network topology irrelevant. If a node is isolated or all of its relevant peers are malicious or unavailable, its local observations can be disrupted. The protocol's broader security argument depends on sufficiently diverse participation, suitable peer connectivity, unpredictable sampling, and the ability of honest nodes to obtain the columns they are supposed to sample.

## 8. Cell KZG proofs and validation

PeerDAS depends on **cell KZG proofs** to authenticate a small data cell against the blob's KZG commitment. This lets a node check an individual cell rather than requiring the entire blob to be present before the node can verify the piece it received.

Proof generation and verification have different cost profiles. The EIP explains that computing cell proofs can be expensive, especially if a producer had to repeatedly compute them for many blobs on the critical path of block production. PeerDAS therefore requires blob transaction senders to compute the required proofs and include them in the transaction-pool wrapper, so clients can use them in the extension and data-column construction pipeline. Verification is designed to be substantially cheaper than proof computation.

A client must not treat the mere presence of a sidecar as sufficient evidence that it is valid. It needs to check that the sidecar is well-formed, that its commitments inclusion proof is correct, and that the cell KZG proofs match the expected commitments and positions. The consensus specifications define the exact verification procedures, containers, and error behavior.

For client developers, this implies a clean separation among:
- sidecar parsing and structural validation;
- inclusion-proof verification;
- cell KZG proof verification;
- availability sampling and retry logic;
- custody storage and serving; and
- reconstruction when enough columns are obtained.

Implementing only one of these pieces is not equivalent to implementing PeerDAS safely.

## 9. Security model and failure modes

PeerDAS is designed to make data withholding detectable while reducing each node's download burden. Its security argument depends on several properties working together.

**Data withholding.** A block producer may publish commitments while failing to provide enough underlying data. Erasure coding is meant to spread the effect of withheld data across many extended pieces, and independently chosen samples make the missing pieces more likely to be requested by honest nodes.

**Invalid cell or proof.** A peer could send a malformed column or a cell that does not open against the expected commitment. Authentication checks must reject data that fails the required KZG or inclusion proofs. Such failure is different from a network timeout.

**Peer isolation and correlated failures.** A node with a poor or adversarial peer set may not receive the columns it needs. Peer diversity and retry behavior are practical parts of the design, not a cosmetic optimization.

**Predictable sample selection.** If a producer could know and selectively serve all requested samples while withholding the rest, sampling would lose much of its power. The scheme's unpredictable pseudo-random selection and distributed participation underpin its detection properties.

**Insufficient redundancy or incorrect implementation.** Erasure coding, cell construction, commitment indexing, and reconstruction thresholds must match the specification. Bugs in matrix indexing or proof association can undermine sampling even when the high-level concept is sound.

**Confusing sample success with full availability.** One node receiving its own chosen columns does not prove it has reconstructed the entire data set. The global inference relies on the protocol's statistical argument and participation assumptions.

**Data retention assumptions.** PeerDAS is for availability during the protocol's relevant time window, not indefinite archiving. It is designed to help honest participants receive and reconstruct data while the protocol requires it; historical availability beyond the retention period is a different service.

## 10. What PeerDAS changes for rollups

PeerDAS increases the feasible blob data capacity by reducing the amount of data each node must download to perform its availability check. This helps Ethereum scale the data resource used by rollups. More data capacity can reduce the data-publication cost pressure passed through to L2 users, although end-user fees still depend on the active blob fee market, execution costs, rollup batching and compression, and operator pricing.

PeerDAS itself does not decide a rollup's state transition, prove an L2 transaction valid, or remove the need for an appropriate settlement and bridge design. It improves the mechanism by which Ethereum's blob data is distributed and checked. Rollups still need to publish all information required by their protocol, and users still need to understand the security assumptions of the rollup's proof system and bridge.

Developers building L2 fee estimators should treat PeerDAS and the blob fee market as related but separate layers. PeerDAS changes the network's data-availability mechanism and enables capacity scaling; the blob fee market prices blob gas. A developer should not infer a specific fee from the fact that PeerDAS is enabled.

## 11. Developer and node-operator checklist

When reading or implementing PeerDAS, verify the following:

- Which consensus fork and configuration does the client target?
- Are cells, columns, sidecars, commitments, and proofs indexed exactly as the consensus specification requires?
- Does the client validate sidecar structure, inclusion proofs, and cell KZG proofs before using received samples?
- Does sampling implement the active SAMPLES_PER_SLOT and custody-group rules?
- Is custody-group selection deterministic and consistent with the node's advertised discovery metadata?
- Does the node join and serve the correct column subnets?
- Can it fetch missing columns using the specified request/response protocol when gossip is insufficient?
- Does the implementation distinguish malformed data from temporary networking failures?
- Are retry policies and peer selection designed to avoid overdependence on one peer?
- Can the client reconstruct the data matrix when enough valid columns are available?
- Does monitoring distinguish sampling success, custody status, sidecar validation failures, missing data, and reconstruction?
- Are protocol parameters read from the active configuration instead of hard-coded across upgrades?

## 12. Key takeaways

- PeerDAS is Ethereum's concrete implementation of Data Availability Sampling for blobs.
- Erasure coding extends blobs; the extended data is divided into authenticated cells and arranged into columns.
- Nodes sample columns to check availability and custody deterministic subsets to serve other peers.
- Cell KZG proofs let clients authenticate small pieces against blob commitments.
- Gossip distributes columns, peer discovery helps find custodians, and request/response can fetch missing columns.
- Sampling and full reconstruction have different goals; local sample success is not the same as reconstructing every blob.
- The active protocol configuration matters. Column counts, sample requirements, custody rules, and related parameters must be checked against the relevant consensus specs.
- PeerDAS supports higher blob capacity, but it is not a validity proof, a permanent archive, or a replacement for rollup settlement and bridge security.

## References

1. [EIP-7594 — PeerDAS: Peer Data Availability Sampling](https://eips.ethereum.org/EIPS/eip-7594)
2. [Ethereum.org — PeerDAS](https://ethereum.org/roadmap/fusaka/peerdas/)
3. [Ethereum Consensus Specifications — DAS Core](https://ethereum.github.io/consensus-specs/specs/fulu/das-core/)
4. [Ethereum Consensus Specifications — Fulu Validator](https://ethereum.github.io/consensus-specs/specs/fulu/validator/)
5. [Ethereum Consensus Specifications — Fulu P2P Interface](https://ethereum.github.io/consensus-specs/specs/fulu/p2p-interface/)
6. [Ethereum Foundation — Fusaka Mainnet Announcement](https://blog.ethereum.org/2025/11/06/fusaka-mainnet-announcement)
7. [EIP-4844 — Shard Blob Transactions](https://eips.ethereum.org/EIPS/eip-4844)
8. [Ethereum.org — Data Availability](https://ethereum.org/developers/docs/data-availability/)
