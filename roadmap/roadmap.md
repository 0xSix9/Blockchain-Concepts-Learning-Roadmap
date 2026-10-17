# Full Learning Roadmap

This is the canonical checklist for the learning plan. It follows the 10 stages and four priority groups in the original roadmap.

- [Priority 1 — Stages 01–04](learning-priorities.md#priority-1--stages-01-to-04)
- [Priority 2 — Stages 05–06](learning-priorities.md#priority-2--stages-05-to-06)
- [Priority 3 — Stages 07–08](learning-priorities.md#priority-3--stages-07-to-08)
- [Priority 4 — Stages 09–10](learning-priorities.md#priority-4--stages-09-to-10)

> Each item is a learning topic. Published articles are linked from the checklist; topics without an article link are still waiting to be documented.

## Stage 01 — EVM Execution

**Priority 1** · [Stage index](../concepts/01-evm-execution/README.md)

- [ ] [EVM Opcodes](../concepts/01-evm-execution/01-evm-opcodes.md) — article published
- [ ] [Stack, Memory, and Calldata](../concepts/01-evm-execution/02-stack-memory-and-calldata.md) — article published
- [ ] [Execution Context](../concepts/01-evm-execution/03-execution-context.md) — article published
- [ ] [CALL](../concepts/01-evm-execution/04-call.md) — article published
- [ ] [DELEGATECALL](../concepts/01-evm-execution/05-delegatecall.md) — article published
- [ ] [STATICCALL](../concepts/01-evm-execution/06-staticcall.md) — article published
- [ ] [CREATE](../concepts/01-evm-execution/07-create.md) — article published
- [ ] [CREATE2](../concepts/01-evm-execution/08-create2.md) — article published
- [ ] [Call Stack and Call Depth](../concepts/01-evm-execution/09-call-stack-and-call-depth.md) — article published
- [ ] [Return Data and Revert Data](../concepts/01-evm-execution/10-return-data-and-revert-data.md) — article published
- [ ] Contract Creation and Initialization
- [ ] State Transition Function

## Stage 02 — ABI and Contract Interaction

**Priority 1** · [Stage index](../concepts/02-abi-contract-interaction/README.md)

- [ ] Application Binary Interface (ABI)
- [ ] ABI Encoding and Decoding
- [ ] Function Selectors
- [ ] Function Dispatch
- [ ] External Function Calls
- [ ] Constructors
- [ ] Fallback Functions
- [ ] Receive Functions
- [ ] Visibility
- [ ] Mutability
- [ ] Payable Functions
- [ ] Events and Event Logs
- [ ] Topics and Indexed Parameters
- [ ] Custom Errors
- [ ] Error Selectors
- [ ] Precompiled Contracts

## Stage 03 — Storage and Data Layout

**Priority 1** · [Stage index](../concepts/03-storage-data-layout/README.md)

- [ ] Storage Layout
- [ ] Storage Slots
- [ ] Storage Packing
- [ ] Mappings and Dynamic Arrays in Storage
- [ ] Memory Layout
- [ ] Calldata Layout
- [ ] Transient Storage (EIP-1153)
- [ ] Storage Context
- [ ] Storage Collisions
- [ ] Recursive Length Prefix (RLP) Encoding
- [ ] Merkle Proofs
- [ ] Account Proofs
- [ ] Storage Proofs

## Stage 04 — Smart Contract Architecture

**Priority 1** · [Stage index](../concepts/04-smart-contract-architecture/README.md)

- [ ] Contract Composition
- [ ] Inheritance
- [ ] Interfaces and Libraries
- [ ] Internal and External Calls
- [ ] Proxy Contracts
- [ ] Transparent Proxies
- [ ] UUPS Proxies
- [ ] Beacon Proxies
- [ ] Minimal Proxies (Clones)
- [ ] Delegatecall-Based Proxy Architecture
- [ ] Contract Upgradeability
- [ ] Storage Compatibility
- [ ] Initializers
- [ ] Storage Collisions in Proxies
- [ ] Diamond Pattern (EIP-2535)
- [ ] Deterministic Contract Deployment
- [ ] CREATE2 Address Derivation

## Stage 05 — Advanced Transactions and Gas

**Priority 2** · [Stage index](../concepts/05-advanced-transactions-gas/README.md)

- [ ] Transaction Lifecycle
- [ ] Transaction Nonce
- [ ] Transaction Types (EIP-2718)
- [ ] Access List Transactions (EIP-2930)
- [ ] Set Code Transactions (EIP-7702)
- [ ] Chain ID and Replay Protection
- [ ] Intrinsic Gas
- [ ] Gas Accounting
- [ ] Gas Limit and Gas Used
- [ ] Out-of-Gas
- [ ] Transaction Receipts
- [ ] Transaction Reverts
- [ ] Transaction Replacement
- [ ] Gas Refunds

## Stage 06 — Application Connectivity

**Priority 2** · [Stage index](../concepts/06-application-connectivity/README.md)

- [ ] JSON-RPC
- [ ] Ethereum RPC Methods
- [ ] Reading Blockchain State
- [ ] Sending Transactions
- [ ] Transaction Signing
- [ ] eth_call
- [ ] eth_estimateGas
- [ ] eth_sendRawTransaction
- [ ] Log and Event Filtering
- [ ] WebSocket Subscriptions
- [ ] Execution Clients
- [ ] Archive Nodes
- [ ] Full Nodes
- [ ] Light Clients
- [ ] Execution API
- [ ] Consensus API
- [ ] Engine API

## Stage 07 — Advanced Layer 2 and Cross-Chain

**Priority 3** · [Stage index](../concepts/07-layer2-cross-chain/README.md)

- [ ] Fraud Proofs
- [ ] Validity Proofs
- [ ] Sequencers
- [ ] Batch Submission
- [ ] State Commitments
- [ ] Settlement Layer
- [ ] Challenge Periods
- [ ] Forced Inclusion
- [ ] Forced Withdrawals
- [ ] Rollup Bridges
- [ ] Blob Fee Market
- [ ] Data Availability Sampling
- [ ] PeerDAS
- [ ] Validiums
- [ ] Volitions
- [ ] Based Rollups
- [ ] Shared Sequencing
- [ ] Cross-Chain Messaging
- [ ] Bridge Security Models

## Stage 08 — Mempool and MEV

**Priority 3** · [Stage index](../concepts/08-mempool-mev/README.md)

- [ ] Transaction Pool
- [ ] Mempool
- [ ] Transaction Propagation
- [ ] Transaction Ordering
- [ ] Front-Running
- [ ] Back-Running
- [ ] Sandwich Attacks
- [ ] Maximal Extractable Value (MEV)
- [ ] Searchers
- [ ] Block Builders
- [ ] Private Transaction Order Flow
- [ ] Private Order Flow Auctions
- [ ] Proposer-Builder Separation (PBS)
- [ ] MEV-Boost
- [ ] Encrypted Mempools
- [ ] Censorship Resistance

## Stage 09 — Advanced Ethereum Consensus

**Priority 4** · [Stage index](../concepts/09-ethereum-consensus/README.md)

- [ ] Slots and Epochs
- [ ] Validator Lifecycle
- [ ] Attestations
- [ ] Proposer Selection
- [ ] Fork Choice Rule
- [ ] LMD-GHOST
- [ ] Casper FFG
- [ ] Checkpoints
- [ ] Finality
- [ ] Slashing
- [ ] Inactivity Leak
- [ ] Sync Committees
- [ ] Chain Reorganization
- [ ] Weak Subjectivity
- [ ] Economic Security
- [ ] Execution Layer and Consensus Layer
- [ ] Consensus Client and Execution Client Communication
- [ ] Ethereum Improvement Proposals (EIPs)

## Stage 10 — Advanced Cryptography and Protocol Developments

**Priority 4** · [Stage index](../concepts/10-advanced-cryptography-protocol/README.md)

- [ ] Signature Recovery
- [ ] BLS Signatures
- [ ] KZG Commitments
- [ ] Polynomial Commitments
- [ ] Verkle Trees
- [ ] Stateless Clients
- [ ] State Expiry
- [ ] Account Abstraction
- [ ] ERC-4337 Architecture
- [ ] EIP-2537
- [ ] Restaking
- [ ] Shared Security
- [ ] Advanced Proposer-Builder Separation (PBS)
- [ ] Advanced MEV Mitigation
- [ ] Future Ethereum Data Availability Designs

---

[Back to project README](../README.md)
