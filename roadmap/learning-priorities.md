# Learning Priorities

The original roadmap groups the 10 stages into four priorities. This is a suggested learning order—not a claim that later topics are unimportant.

## Priority 1 — Stages 01 to 04

**Focus:** EVM execution, ABI and contract interaction, storage and data layout, and smart-contract architecture.

These stages establish the closest foundations for understanding how contracts execute, how calls are encoded, how state is laid out, and how contracts compose or use proxy patterns.

- [Stage 01 — EVM Execution](../concepts/01-evm-execution/README.md)
- [Stage 02 — ABI and Contract Interaction](../concepts/02-abi-contract-interaction/README.md)
- [Stage 03 — Storage and Data Layout](../concepts/03-storage-data-layout/README.md)
- [Stage 04 — Smart Contract Architecture](../concepts/04-smart-contract-architecture/README.md)

## Priority 2 — Stages 05 to 06

**Focus:** transaction mechanics, gas, RPC, and application connectivity.

This group covers transaction lifecycle and types, gas accounting, receipts and reverts, signing and submission, JSON-RPC, clients, and APIs.

- [Stage 05 — Advanced Transactions and Gas](../concepts/05-advanced-transactions-gas/README.md)
- [Stage 06 — Application Connectivity](../concepts/06-application-connectivity/README.md)

## Priority 3 — Stages 07 to 08

**Focus:** Layer 2, cross-chain architecture, mempool mechanics, and MEV.

This group introduces scaling and settlement models, data availability, bridge security, transaction ordering, block building, private order flow, PBS, and encrypted mempools.

- [Stage 07 — Advanced Layer 2 and Cross-Chain](../concepts/07-layer2-cross-chain/README.md)
- [Stage 08 — Mempool and MEV](../concepts/08-mempool-mev/README.md)

## Priority 4 — Stages 09 to 10

**Focus:** Ethereum consensus, advanced cryptography, and protocol developments.

These topics deepen understanding of proof-of-stake consensus, validator incentives, finality, cryptographic commitments, statelessness, account abstraction, shared security, and future protocol designs.

- [Stage 09 — Advanced Ethereum Consensus](../concepts/09-ethereum-consensus/README.md)
- [Stage 10 — Advanced Cryptography and Protocol Developments](../concepts/10-advanced-cryptography-protocol/README.md)

## How to interpret the priorities

1. Start with Priority 1 if your immediate goal is to understand smart-contract execution and architecture.
2. Move to Priority 2 to understand transactions and application-to-node communication.
3. Explore Priority 3 for scaling, cross-chain systems, transaction ordering, and MEV.
4. Study Priority 4 progressively, revisiting earlier concepts whenever needed.

You do not need to master every advanced topic before building useful contracts. Pair reading with experiments, implementation, debugging, and primary-source research.

[Full checklist](roadmap.md) · [Back to README](../README.md)
