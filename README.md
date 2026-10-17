<div align="center">
  <img src="assets/roadmap-overview.svg" alt="Blockchain Concepts Learning Roadmap: ten stages from EVM execution to advanced protocol developments" width="100%" />
  <br/>
  <p>
    <a href="roadmap/roadmap.md"><img src="https://img.shields.io/badge/ROADMAP-10%20STAGES-2563eb?style=for-the-badge" alt="10 stages" /></a>
    <a href="roadmap/learning-priorities.md"><img src="https://img.shields.io/badge/LEARNING-4%20PRIORITIES-0891b2?style=for-the-badge" alt="4 priorities" /></a>
    <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-16a34a?style=for-the-badge" alt="MIT License" /></a>
  </p>
  <h3>A practical map of the concepts behind Ethereum, smart contracts, and blockchain infrastructure.</h3>
  <p>Created and maintained by <a href="https://github.com/0xSix9">0xSix9</a>.</p>
</div>

---

## What is this?

**Blockchain Concepts Learning Roadmap** is a structured, developer-oriented learning plan for understanding how Ethereum and EVM-based applications work beneath the surface.

The roadmap starts with EVM execution and contract interaction, then moves through storage, contract architecture, transactions, gas, RPC, Layer 2, cross-chain systems, MEV, consensus, and advanced cryptography.

This repository is designed as both a **learning checklist** and a **growing technical reference**. Every stage has its own index, and the complete roadmap keeps all concepts in one place.

> **Publication progress:** ✅ **Stage 01 — EVM Execution is content-complete:** all 12 concept articles are published and reviewed. **Stage 02 — ABI and Contract Interaction is in progress:** the Application Binary Interface (ABI), ABI Encoding and Decoding, Function Selectors, Function Dispatch, External Function Calls, Constructors, Fallback Functions, Receive Functions, Visibility, Mutability, and Payable Functions articles are published, and the remaining topics will be documented incrementally.

## Start here

- **[Full roadmap — all 157 concepts](roadmap/roadmap.md)** — browse the complete checklist.
- **[EVM Opcodes](concepts/01-evm-execution/01-evm-opcodes.md)**, **[Stack, Memory, and Calldata](concepts/01-evm-execution/02-stack-memory-and-calldata.md)**, **[Execution Context](concepts/01-evm-execution/03-execution-context.md)**, **[CALL](concepts/01-evm-execution/04-call.md)**, **[DELEGATECALL](concepts/01-evm-execution/05-delegatecall.md)**, **[STATICCALL](concepts/01-evm-execution/06-staticcall.md)**, **[CREATE](concepts/01-evm-execution/07-create.md)**, **[CREATE2](concepts/01-evm-execution/08-create2.md)**, **[Call Stack and Call Depth](concepts/01-evm-execution/09-call-stack-and-call-depth.md)**, **[Return Data and Revert Data](concepts/01-evm-execution/10-return-data-and-revert-data.md)**, **[Contract Creation and Initialization](concepts/01-evm-execution/11-contract-creation-and-initialization.md)**, and **[State Transition Function](concepts/01-evm-execution/12-state-transition-function.md)** — all 12 published Stage 01 articles.
- **[Application Binary Interface (ABI)](concepts/02-abi-contract-interaction/01-application-binary-interface-abi.md)** — the interface schema, selectors, and contract-interaction overview.
- **[ABI Encoding and Decoding](concepts/02-abi-contract-interaction/02-abi-encoding-and-decoding.md)** — static and dynamic types, head/tail, offsets, padding, and packed-encoding pitfalls.
- **[Function Selectors](concepts/02-abi-contract-interaction/03-function-selectors.md)** — canonical signatures, Keccak-256, calldata dispatch, overloads, and collision risks.
- **[Function Dispatch](concepts/02-abi-contract-interaction/04-function-dispatch.md)** — how contract bytecode routes calldata, handles unmatched selectors, and distinguishes fallback from receive.
- **[External Function Calls](concepts/02-abi-contract-interaction/05-external-function-calls.md)** — cross-contract calls, CALL/STATICCALL/DELEGATECALL, return and revert data, gas, and reentrancy.
- **[Constructors](concepts/02-abi-contract-interaction/06-constructors.md)** — one-time initialization, constructor arguments, inheritance, immutables, proxy initialization, and deployment risks.
- **[Fallback Functions](concepts/02-abi-contract-interaction/07-fallback-functions.md)** — handling unmatched calldata, empty-calldata rules, payable behavior, raw return bytes, and proxy routing.
- **[Receive Functions](concepts/02-abi-contract-interaction/08-receive-functions.md)** — empty-calldata ETH reception, gas constraints, accounting caveats, and security considerations.
- **[Visibility](concepts/02-abi-contract-interaction/09-visibility.md)** — function and state-variable access scope, inheritance, ABI exposure, and the difference between visibility, privacy, and authorization.
- **[Mutability](concepts/02-abi-contract-interaction/10-mutability.md)** — pure, view, nonpayable, and payable behavior; state reads and writes; Ether acceptance; and STATICCALL.
- **[Payable Functions](concepts/02-abi-contract-interaction/11-payable-functions.md)** — receiving ETH, validating `msg.value`, accounting, receive/fallback paths, and safe outgoing transfers.
- **[Learning priorities](roadmap/learning-priorities.md)** — understand the four-stage groups and suggested order.
- **[Concept article template](templates/concept-template.md)** — a starting structure for writing each article later.
- **[Sources and research policy](references/sources.md)** — recommended primary sources and documentation rules.
- **[Contributing guide](CONTRIBUTING.md)** — naming, quality, and review conventions.
- **[Changelog](CHANGELOG.md)** — notable repository changes.

## Roadmap at a glance

| Stage | Focus | Concepts | Priority | Article publication |
|---|---|---:|:---:|---|
| [✅ Stage 01 — EVM Execution](concepts/01-evm-execution/README.md) | EVM execution fundamentals | 12 | P1 | ✅ Complete — 12/12 published |
| [Stage 02 — ABI and Contract Interaction](concepts/02-abi-contract-interaction/README.md) | ABI, selectors, calls, events, errors | 16 | P1 | In progress — 11/16 published |
| [Stage 03 — Storage and Data Layout](concepts/03-storage-data-layout/README.md) | Storage slots, packing, mappings, proofs | 13 | P1 | Checklist only |
| [Stage 04 — Smart Contract Architecture](concepts/04-smart-contract-architecture/README.md) | Composition, proxies, upgradeability | 17 | P1 | Checklist only |
| [Stage 05 — Advanced Transactions and Gas](concepts/05-advanced-transactions-gas/README.md) | Transaction mechanics and gas accounting | 14 | P2 | Checklist only |
| [Stage 06 — Application Connectivity](concepts/06-application-connectivity/README.md) | RPC, clients, APIs, event subscriptions | 17 | P2 | Checklist only |
| [Stage 07 — Advanced Layer 2 and Cross-Chain](concepts/07-layer2-cross-chain/README.md) | Rollups, bridges, data availability | 19 | P3 | Checklist only |
| [Stage 08 — Mempool and MEV](concepts/08-mempool-mev/README.md) | Transaction ordering and MEV | 16 | P3 | Checklist only |
| [Stage 09 — Advanced Ethereum Consensus](concepts/09-ethereum-consensus/README.md) | Validators, fork choice, finality | 18 | P4 | Checklist only |
| [Stage 10 — Advanced Cryptography and Protocol Developments](concepts/10-advanced-cryptography-protocol/README.md) | Cryptography and future protocol designs | 15 | P4 | Checklist only |

**157 roadmap entries across 10 stages.** This count represents topics in the learning plan, not completed articles.

## Learning priorities

| Priority | Stages | Focus |
|---|---|---|
| **P1** | 01–04 | Core foundations: EVM execution, contract interaction, storage, and smart-contract architecture. |
| **P2** | 05–06 | Transactions, gas accounting, RPC, clients, and application connectivity. |
| **P3** | 07–08 | Layer 2, cross-chain architecture, mempool mechanics, transaction ordering, and MEV. |
| **P4** | 09–10 | Ethereum consensus, advanced cryptography, and protocol-level developments. |

Priority describes the suggested learning order, not the overall importance of a topic. The first four stages establish the most direct foundations for smart-contract development; later stages broaden protocol and infrastructure understanding.

## How to use the roadmap

1. Open a stage index and choose a concept.
2. Research it using primary sources before writing.
3. Explain the mechanism in your own words and connect it to developer use cases.
4. Add a diagram, code experiment, or trace when it genuinely improves understanding.
5. Record references and review dates, especially for protocol topics that evolve.
6. Mark a topic complete only after the article has been checked.

### Status convention

A ✅ beside a **stage** indicates article-publication progress. The unchecked boxes beside individual concepts are separate study checklists and should be marked only after reviewing those concepts.

- ⬜ **Not started** — checklist only; no article yet.
- 🟡 **In progress** — research or drafting is underway.
- ✅ **Reviewed** — article is written and checked against its references.
- 🔄 **Needs update** — a specification, implementation, or source may have changed.

## Repository structure

```text
.
├── README.md
├── roadmap/
│   ├── roadmap.md
│   └── learning-priorities.md
├── concepts/
│   ├── 01-evm-execution/
│   ├── 02-abi-contract-interaction/
│   ├── 03-storage-data-layout/
│   ├── 04-smart-contract-architecture/
│   ├── 05-advanced-transactions-gas/
│   ├── 06-application-connectivity/
│   ├── 07-layer2-cross-chain/
│   ├── 08-mempool-mev/
│   ├── 09-ethereum-consensus/
│   └── 10-advanced-cryptography-protocol/
├── templates/
├── references/
├── assets/
│   ├── diagrams/
│   ├── images/
│   └── roadmap-overview.svg
└── .github/
    ├── ISSUE_TEMPLATE/
    └── PULL_REQUEST_TEMPLATE.md
```

## Project principles

- **Concept-first:** explain mechanisms, not just definitions.
- **Developer-oriented:** connect ideas to implementation, debugging, architecture, gas, and security.
- **Traceable:** support technical claims with relevant sources.
- **Visual where useful:** diagram execution flows, data layouts, and protocol boundaries when they clarify the idea.
- **Incremental:** empty checklists are not presented as finished content.
- **Maintainable:** update documentation when relevant specifications or protocol behavior change.

## Scope and accuracy

The roadmap is Ethereum/EVM-centered. Some ideas apply more broadly, but behavior can vary by chain, client, fork, standard, or version. Future articles should clearly distinguish protocol rules from client-specific behavior and common practice.

This is an independent educational project, not official Ethereum documentation, an audit, or a security guarantee.

---

<div align="center"><sub>Learn the mechanism. Trace the execution. Understand the protocol.</sub></div>
