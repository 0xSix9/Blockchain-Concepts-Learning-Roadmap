# Data Availability Sampling (DAS)

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

![Data Availability Sampling overview: a producer erasure-codes a dataset, distributes it across a peer network, and independent nodes sample random pieces. Successful sampling gives probabilistic confidence that enough data is available to reconstruct the original dataset.](../../assets/diagrams/12-data-availability-sampling.svg)

## 1. Overview

**Data Availability Sampling (DAS)** is a technique for checking whether a block's data has been published and can be retrieved, without requiring every node to download the entire dataset. Instead, nodes independently request small, randomly selected parts of the data. If the data has been encoded with suitable redundancy and a node can retrieve its assigned samples, those observations provide probabilistic confidence that the complete dataset is available.

DAS addresses an important scaling limit. Rollups need to publish data so that independent participants can reconstruct their state transitions, verify their behavior, and challenge invalid claims. If every validator and node must download every byte as the data payload grows, bandwidth requirements grow alongside throughput. That can make it harder for ordinary participants to operate nodes, weakening decentralization.

Sampling changes how the workload is distributed. Rather than making every node download all of the data, the protocol divides and redundantly encodes the dataset, then distributes pieces across many peers. Each participating node checks a subset. When sampling is properly designed, unavailable data is likely to cause one or more samplers to fail, revealing that the proposed block cannot safely be treated as available.

DAS is not a magic proof that every byte has been stored forever. It is a protocol for checking present data availability under explicit assumptions about encoding, sampling, network participation, and peer behavior. It also does not prove that the state transition is valid; validity and data availability are separate properties.

## 2. What does data availability mean?

**Data availability** is confidence that the information needed to independently verify a block or reconstruct an application state has actually been published and can be obtained. For a rollup, this usually means transaction data and other inputs required for independent derivation or dispute procedures—not merely a transaction hash or a statement from the sequencer.

Data availability matters because a commitment to data is not the same as the data itself. A producer can publish a commitment that cryptographically identifies a payload while withholding the payload. The commitment may still be well-formed, but users and verifiers cannot reconstruct what they need from a commitment alone.

Consider a rollup batch. The batcher publishes a compressed list of transactions and related information. A node that can obtain and process the data can independently derive the rollup state or verify a proof or challenge. If the batcher posts a commitment and hides the underlying data, independent participants might be unable to reconstruct the state or exercise the rollup's dispute procedure. The chain can contain a commitment to something that other participants cannot inspect.

DAS helps answer a focused question: **Is the encoded dataset available across the network now, to a high degree of confidence?** It does not answer every question about the data's correctness, execution semantics, or long-term archival.

## 3. The core idea: erasure coding plus random sampling

DAS combines two ideas.

### Erasure coding creates redundancy

A dataset is encoded into a larger representation with redundant pieces. The encoding is designed so the original data can be reconstructed even when some encoded pieces are missing. This is different from simply copying the dataset several times: a well-designed erasure code can distribute recoverability across pieces rather than relying on complete replicas.

A common conceptual model doubles the encoded representation. If the data is encoded into an expanded set of evaluations, the original can be reconstructed from a sufficiently large subset—often described, in this simplified model, as at least half of the extended data. If a producer withholds a part of the original dataset, the corresponding encoding constraints mean a substantial portion of the expanded representation cannot be provided consistently. The exact construction and recovery threshold depend on the protocol.

The important effect is **amplification**: hiding a small amount of original data should not allow a producer to hide only a tiny, undetectable fraction of the coded representation. Proper erasure coding makes withholding show up across many samples.

### Sampling asks for only a subset

Instead of downloading every encoded piece, each node requests a number of randomly chosen samples from peers. The node checks that those samples were actually received and satisfy the protocol's authentication rules. Different nodes sample different pieces, so a producer who withholds data is more likely to be caught by at least one honest participant as the number and diversity of samples increase.

Sampling must be unpredictable enough that a malicious producer cannot know in advance which exact pieces every verifier will request and selectively serve only those pieces while hiding the rest. Protocols therefore need appropriate randomness, authenticated sample data, peer discovery, and rules for how responses are obtained and validated.

The two components work together: erasure coding makes withholding affect many encoded pieces; sampling makes the network check only a fraction of those pieces.

## 4. A simplified probability intuition

Suppose, for intuition only, that a withholding attack leaves half of the expanded coded pieces unavailable and each sample is independent and uniformly random. The probability that one sample accidentally lands on an available piece is one half. The probability that all (k) samples miss the unavailable pieces is then approximately:

[
P(	ext{miss}) = left(rac{1}{2}ight)^k
]

So the corresponding probability of detecting at least one missing sample is:

[
P(	ext{detect}) = 1 - left(rac{1}{2}ight)^k
]

For example, with 10 independent samples in this simplified scenario, the chance of missing every unavailable piece is about (1/1024), or 0.098%. More samples reduce this probability quickly.

This is an intuition aid, **not a universal DAS security formula**. Real protocol analyses must account for the precise erasure code, how much data the producer withholds, sample construction, correlation between samples, peer topology, adversarial strategies, and the probability model for honest participants. A concrete protocol may sample columns, cells, or other authenticated units rather than independent single pieces. Its specified parameters and security analysis—not the toy formula—determine the actual guarantee.

The lesson is not that a particular sample count automatically makes a network secure. It is that the detection probability can rise rapidly with more independent samples, provided the encoding and sampling assumptions hold.

## 5. How a DAS protocol works

A generic data availability sampling design can be described as a pipeline:

1. **Commit to a dataset.** The producer constructs the payload and a cryptographic commitment or authenticated representation that allows verifiers to check the pieces they receive.
2. **Encode with redundancy.** The protocol expands the data into pieces that preserve enough information to reconstruct the original.
3. **Distribute the pieces.** Pieces are propagated or assigned across a peer network. The design must make it possible to discover which peers can supply each requested piece.
4. **Choose unpredictable samples.** Each sampling node selects a subset according to the protocol's randomness and sampling rules.
5. **Request and authenticate the pieces.** The node obtains samples from peers and verifies them against the commitment and the protocol's coding rules.
6. **Record the local result.** Successful retrieval supports a local availability check; failed requests or invalid responses can trigger retries, peer penalties, or protocol-specific rejection behavior.
7. **Combine the network's checks.** The security argument relies on enough honest, appropriately connected participants sampling independently so that withholding is likely to be detected before unavailable data is accepted as available.

The exact responsibilities differ by design. A protocol may distribute pieces through gossip, use specific peer requests, assign data custody deterministically, or rely on committee roles. DAS is the general strategy; the networking protocol determines how that strategy is carried out in production.

## 6. DAS, validity, and data commitments are different

Several concepts are related but should not be confused:

- **Data availability** asks whether the data necessary for verification or reconstruction can be obtained.
- **Validity** asks whether a proposed state transition follows the protocol rules. A validity proof can show that a transition satisfies a statement, but a proof by itself does not necessarily publish all data users need.
- **A data commitment** identifies a dataset or binds a producer to a representation. A commitment alone does not prove the data has been released to the network.
- **Data integrity** asks whether retrieved pieces match the committed dataset. Authentication helps prevent peers from substituting arbitrary pieces, but integrity of received samples is not alone a guarantee that all unreceived pieces exist.
- **Finality** concerns whether the containing block or state can be reverted under the consensus protocol. It is not synonymous with data availability.
- **Archival** concerns retaining data over longer periods. DAS checks availability under the protocol's rules; it does not require every node to keep every piece forever.

These distinctions matter when evaluating a rollup. A system can have a correct validity proof yet provide weak data availability. A system can also publish all required data while proposing invalid state transitions. Security analysis must assess both dimensions and the system's settlement and bridge assumptions.

## 7. DAS versus full replication and data availability committees

### Full replication

In a fully replicated design, each full node downloads and validates all relevant data. This provides a direct local observation: if the node has the whole payload, it can verify that the payload was available to it. The approach is conceptually simple but makes per-node bandwidth and data handling scale with the full dataset.

DAS aims to reduce the per-node data burden. Each sampler fetches less data, while the network's collective set of independent checks supports confidence in availability. This advantage comes with more protocol complexity: erasure coding, authenticated samples, sampling randomness, peer discovery, and network assumptions all need careful engineering.

### Data availability committees (DACs)

A data availability committee is a designated group that stores or checks data and provides attestations about availability. The guarantee depends on committee size, membership, honesty assumptions, incentives, and what happens when members fail or collude.

DAS and DACs therefore have different trust models. DAS tries to distribute checks across many independent participants and use randomized sampling plus coding to make withholding detectable. A committee-based model relies more directly on the behavior and accountability of its designated members. Some systems combine availability attestations with other mechanisms; labels alone do not determine the security model.

DAS is not automatically more secure in every deployment. Participation, network connectivity, sampling implementation, erasure-code correctness, and the assumptions behind acceptance rules all matter.

## 8. Ethereum and PeerDAS: general idea versus implementation

Ethereum introduced blobs with EIP-4844 to provide rollups with a dedicated data-publication resource. Earlier blob handling required nodes to download the complete blob data to establish availability locally. **PeerDAS**, specified by EIP-7594 and activated as part of the Fusaka upgrade on Ethereum Mainnet on December 3, 2025, applies the DAS approach to Ethereum's blob network.

PeerDAS extends blob data using erasure coding and organizes the extended data into cells and columns. Nodes custody assigned subsets of columns, discover peers that hold them, and request samples from those peers. The node need not independently download every complete blob to perform its availability checks. The EIP describes how each node's custody assignment is determined and how sampling interacts with gossip and peer requests.

This is the practical instantiation of the general principle described in this article. **DAS is the broad technique; PeerDAS is Ethereum's particular peer-to-peer protocol for applying it to blobs.** PeerDAS also has exact data structures, custody rules, sample counts, networking behavior, and recovery procedures. Those details are protocol-specific and should be studied in the dedicated [PeerDAS concept](./13-peerdas.md) rather than assumed to apply to every DAS system.

The scaling purpose is to let Ethereum increase blob throughput without requiring each individual node to download all blob data in full. Reducing per-node data-transfer requirements helps keep participation more accessible as aggregate data throughput increases. That scaling goal remains bounded by the protocol's availability design, network conditions, client implementation, and active parameters.

## 9. Security assumptions and failure modes

Sampling is a probabilistic design, so its security depends on more than choosing a random number of bytes.

**Withholding attacks.** A producer may attempt to publish commitments while refusing to reveal enough data for reconstruction. Redundant encoding should make meaningful withholding affect enough encoded units that honest samplers are likely to detect it.

**Weak or predictable randomness.** If an attacker can predict all sampling positions, it may serve the requested subset and conceal other pieces. Sampling procedures must prevent a producer from selectively choosing which pieces appear available to every verifier.

**Correlated samples.** The simple probability model assumes independent random samples. If samples are correlated, or many nodes unknowingly sample the same small area, aggregate detection confidence can be substantially weaker than a naive calculation suggests.

**Peer isolation and eclipse attacks.** Sampling nodes must connect to sufficiently diverse peers. If an attacker controls or isolates a node's relevant peer set, it might return misleading evidence or make honest data unreachable from that node. Peer diversity, discovery, retries, and network-level protections therefore matter.

**Invalid encoding or commitments.** Erasure coding and cryptographic authentication must be implemented correctly. If a node cannot distinguish correctly encoded, committed pieces from malformed or inconsistent data, the sampling result is unreliable.

**False confidence from one node.** A local sampler's success only covers the pieces it requested. The global security argument depends on randomized, distributed sampling and the protocol's conditions for accepting data as available—not an assumption that one successful client has seen the whole payload.

**Availability is time-bounded.** Sampling establishes a protocol-specific confidence that data is available during the relevant period. It does not imply that every sample will be hosted forever or that historical payloads will always be served from ordinary peers.

## 10. What DAS means for rollup and node developers

For rollup developers, DAS provides a path to scaling data publication while reducing the amount of data each node needs to fetch. But application security still depends on selecting an appropriate data availability layer, publishing all the information required for recovery, and understanding what its sampling guarantees actually cover.

When evaluating a rollup or DA system, ask:

- What exact data must users and independent verifiers obtain to reconstruct state or execute a dispute?
- Is the data published on Ethereum, sampled through a separate DA network, or attested to by a committee?
- What erasure code and reconstruction threshold are specified?
- Which units are sampled, how is randomness generated, and what sample authentication is used?
- How many independent samples are taken, and what security analysis supports that parameter?
- How are peers discovered, and what protections address peer isolation or correlated failures?
- What evidence is sufficient for the protocol to treat data as available?
- Who can reconstruct unavailable or lost data, and during what time window?
- Does the DA mechanism establish only availability, or does the system separately prove state-transition validity?
- What upgrade, governance, or parameter changes could alter the security assumptions?

For Ethereum node and tooling developers specifically, distinguish general DAS concepts from active PeerDAS specifications. Avoid embedding sample counts, custody assignments, blob parameters, or networking assumptions from one fork into reusable tooling without checking the current consensus specifications.

## 11. Key takeaways

- DAS checks data availability by having nodes retrieve randomly selected samples instead of downloading every piece.
- Erasure coding adds redundancy so that withholding original data affects enough encoded pieces to be detectable.
- More independent, unpredictable samples can sharply reduce the chance of missing a withholding attack, but real guarantees depend on the protocol's analysis and network assumptions.
- Availability is not the same as validity, integrity, finality, or archival.
- DAS has a different trust model from a designated data availability committee, though neither label alone is a full security assessment.
- PeerDAS is Ethereum's specific sampling protocol for blobs; it implements the general DAS approach using peer distribution, cells, columns, and custody rules.
- The goal is to scale data throughput without requiring every node to download every byte, while preserving enough redundancy and independent checks to detect withholding.

## References

1. [Ethereum.org — Data Availability](https://ethereum.org/developers/docs/data-availability/)
2. [Ethereum.org — PeerDAS](https://ethereum.org/roadmap/fusaka/peerdas/)
3. [EIP-7594 — PeerDAS: Peer Data Availability Sampling](https://eips.ethereum.org/EIPS/eip-7594)
4. [Ethereum Foundation — Fusaka Mainnet Announcement](https://blog.ethereum.org/2025/11/06/fusaka-mainnet-announcement)
5. [EIP-4844 — Shard Blob Transactions](https://eips.ethereum.org/EIPS/eip-4844)
6. [Ethereum.org — Danksharding](https://ethereum.org/roadmap/danksharding/)
