# Changelog

Notable repository changes are recorded here.

## 2026-10-16

- Established a 10-stage roadmap and four learning-priority groups.
- Added a linked checklist of all 157 roadmap topics.
- Added the README roadmap illustration, article template, research references, and contribution guidance.
- Added stage indexes and GitHub templates.

Concept articles are added incrementally; placeholders and checklists do not count as completed articles.

## 2026-10-17

- Published the EVM Opcodes article with its overview diagram.
- Published the Stack, Memory, and Calldata article with a dedicated data-area diagram.
- Linked both articles from the Stage 01 index and canonical roadmap.


## 2026-10-17

- Published the Execution Context article with a custom CALL vs DELEGATECALL diagram.
- Linked Execution Context from the Stage 01 index, full roadmap, and project README.
- Published the CALL opcode article with a custom call-frame diagram.
- Linked CALL from the Stage 01 index, full roadmap, and project README.


## 2026-10-17

- Published the DELEGATECALL article with a custom proxy-context diagram.
- Linked DELEGATECALL from the Stage 01 index, full roadmap, and project README.


## 2026-10-17

- Published the STATICCALL article, covering EIP-214 static execution, Solidity view calls, and STATICCALL versus eth_call.
- Added a custom static-context diagram and linked the article from the Stage 01 index, full roadmap, and project README.


## 2026-10-17

- Published the CREATE opcode article, covering initcode, address derivation, deployment execution, runtime-code deposit, failure behavior, and gas/size constraints.
- Added a custom CREATE deployment-flow diagram and linked the article from the Stage 01 index, full roadmap, and project README.


## 2026-10-17

- Published the CREATE2 opcode article, covering EIP-1014 address derivation, initcode hashing, deterministic deployment, gas costs, failure behavior, and security considerations.
- Added a custom CREATE2 deployment-flow diagram and linked the article from the Stage 01 index, full roadmap, and project README.


## 2026-10-17

- Published the Call Stack and Call Depth article, distinguishing nested execution frames from the EVM operand stack and Solidity's “Stack too deep” compiler error.
- Added a custom diagram showing nested message-call frames, the separate operand stack, the 1024 depth limit, and EIP-150's gas-forwarding rule.
- Linked the article from the Stage 01 index, full roadmap, and project README.


## 2026-10-17

- Published the Return Data and Revert Data article, covering EIP-211's return-data buffer, RETURNDATASIZE, RETURNDATACOPY, REVERT payloads, Solidity error encodings, and safe low-level call handling.
- Added a custom diagram showing successful return bytes and failure data flowing through call-like operations.
- Linked the article from the Stage 01 index, full roadmap, and project README.


## 2026-10-17

- Published the Contract Creation and Initialization article, covering creation transactions, initcode versus runtime bytecode, constructor context, storage and immutable initialization, failure behavior, and protocol size limits.
- Added a custom contract-creation lifecycle diagram and linked the article from the Stage 01 index, full roadmap, and project README.


## 2026-10-17

- Published the State Transition Function article, explaining transaction-level and block-level state transitions, validation versus execution failure, rollback boundaries, gas/fee accounting, receipts, and state roots.
- Added a custom state-transition diagram and linked the article from the Stage 01 index, full roadmap, and project README.
- Completed publication of all 12 Stage 01 concept articles; learning checkboxes remain unchecked for the author's personal study progress.


## 2026-10-17

- Completed a technical consistency review of all 12 Stage 01 concept articles against official Ethereum/Solidity documentation and selected core EIPs; checked local Markdown/SVG links and basic SVG structure.
- Standardized review metadata for articles 07–12 and refreshed obsolete “article not published yet” navigation notes.
- Clarified the CREATE nonce wording, made the DELEGATECALL proxy example's receive path consistent, and improved the explanation of cumulative receipt gas accounting.


## 2026-10-17

- Marked Stage 01 as content-complete (12/12 concept articles published) in the stage index, canonical roadmap, root README roadmap table, and overview SVG.
- Clarified that the stage-level checkmark tracks article publication, while individual concept checkboxes remain for personal study progress.


## 2026-10-17

- Published the Application Binary Interface (ABI) article with a custom diagram explaining the compiler ABI artifact, calldata encoding, contract dispatch, return data, logs, and revert data.
- Linked the article from the Stage 02 index, the canonical roadmap, and the root README; updated Stage 02 publication progress to 1/16.


## 2026-10-17

- Published the ABI Encoding and Decoding article covering 32-byte words, static and dynamic types, head/tail layout, relative offsets, padding, Solidity ABI helpers, ethers v6 coding, and packed-encoding ambiguity.
- Added a custom head/tail diagram and linked the article from the Stage 02 index, canonical roadmap, and root README; updated Stage 02 publication progress to 2/16.


## 2026-10-17

- Published the Function Selectors article explaining canonical signatures, Keccak-256 selector derivation, calldata layout, overloads, compiler-generated dispatch, selector collisions, and security boundaries.
- Added a custom selector-flow diagram and linked the article from the Stage 02 index, canonical roadmap, and root README; updated Stage 02 publication progress to 3/16.


## 2026-10-17

- Published the Function Dispatch article describing Solidity-generated selector routing, argument decoding, unmatched-selector behavior, receive versus fallback paths, internal calls, proxies, and custom routers.
- Added a custom dispatch-flow diagram and linked the article from the Stage 02 index, canonical roadmap, and root README; updated Stage 02 publication progress to 4/16.


## 2026-10-17

- Published the External Function Calls article covering high-level and low-level calls, internal-versus-external boundaries, CALL/STATICCALL/DELEGATECALL semantics, execution context, failure and return data, gas, and reentrancy defenses.
- Added a custom external-call diagram and linked the article from the Stage 02 index, canonical roadmap, and root README; updated Stage 02 publication progress to 5/16.


## 2026-10-17

- Published the Constructors article covering creation-time execution, ABI-encoded constructor arguments, initcode versus runtime bytecode, state variables and immutables, inheritance order, payable deployment, factory ownership, proxy initialization, and deployment-size/gas risks.
- Added a custom constructor deployment diagram and linked the article from the Stage 02 index, canonical roadmap, and root README; updated Stage 02 publication progress to 6/16.


## 2026-10-17

- Published the Fallback Functions article covering unmatched calldata, fallback versus receive, payability, the special raw-bytes return form, proxy routing, and related security pitfalls.
- Added a custom fallback/receive dispatch diagram and linked the article from the Stage 02 index, canonical roadmap, and root README; updated Stage 02 publication progress to 7/16.


## 2026-10-17

- Published the Receive Functions article covering empty-calldata routing, receive versus fallback and payable functions, the legacy 2,300-gas stipend, ETH accounting caveats, forced Ether transfers, and reentrancy considerations.
- Added a custom receive routing diagram and linked the article from the Stage 02 index, canonical roadmap, and root README; updated Stage 02 publication progress to 8/16.

## 2026-10-17

- Published the Visibility article, covering Solidity function and state-variable visibility, getters, inheritance, ABI exposure, and why visibility is not privacy or authorization.
- Added a custom scope diagram and linked Visibility from the Stage 02 index, canonical roadmap, and project README.

## 2026-10-17

- Published the State Mutability article, explaining pure, view, nonpayable, and payable functions, ABI mutability metadata, STATICCALL, payable entry points, and security considerations.
- Added a dedicated state-mutability matrix diagram and linked Mutability from the Stage 02 index, canonical roadmap, and project README; updated Stage 02 publication progress to 10/16.

## 2026-10-17

- Published the Payable Functions article, covering msg.value validation, Ether accounting, named payable functions versus receive/fallback, outgoing calls, and reentrancy-aware withdrawal patterns.
- Added a custom Ether-flow diagram and linked Payable Functions from the Stage 02 index, canonical roadmap, and project README; updated Stage 02 publication progress to 11/16.

## 2026-10-17

- Published the Events and Event Logs article, covering event declarations, emit, log address/topics/data, signature topics, ABI decoding, filtering, reverted execution, and reorganization-aware consumers.
- Added a custom event-log flow diagram and linked the article from the Stage 02 index, canonical roadmap, and project README; updated Stage 02 publication progress to 12/16.

## 2026-10-17

- Published the Topics and Indexed Parameters article, explaining standard event topic layout, dynamic indexed-value hashing, topic-position matching, wildcards, OR alternatives, and eth_getLogs filtering.
- Added a dedicated topic-layout and filtering diagram, linked the article throughout Stage 02 navigation, and updated Stage 02 publication progress to 13/16.

## 2026-10-17

- Published the Custom Errors article, covering structured revert data, selectors and ABI-encoded arguments, string comparison, try/catch decoding, ERC-6093, and error-spoofing risks.
- Added a dedicated custom-error revert-data diagram, linked the article throughout Stage 02 navigation, and updated Stage 02 publication progress to 14/16.
- Documented Solidity compiler-version behavior for custom errors in require, including the 0.8.37 fix for the reported named-argument code-generation bug.

## 2026-10-17

- Published the Error Selectors article, covering canonical error signatures, Keccak-256 derivation, built-in Error(string)/Panic(uint256) selectors, reserved values, selector collisions, and decoding/security considerations.
- Added a custom error-selector flow diagram, linked Error Selectors throughout Stage 02 navigation, and updated publication progress to 15/16.


## 2026-10-17

- Published the Precompiled Contracts article, explaining how special addresses invoke client-implemented operations, the common post-Prague precompile set, fork-specific availability, low-level call handling, gas behavior, and security checks.
- Added a custom Precompiled Contracts overview diagram and linked the article from the Stage 02 index, canonical roadmap, and project README.
- Completed Stage 02 publication: all 16 of 16 concept articles are now published; the personal study checkboxes remain unchecked.


## 2026-10-17

- Audited all 16 Stage 02 articles for conceptual consistency, code-example plausibility, official references, diagrams, and navigation; checked 59 internal Markdown link instances across 28 unique local targets with no missing destinations.
- Corrected the Precompiled Contracts article and diagram to include Ethereum mainnet's Osaka `P256VERIFY` precompile at `0x0100`, and clarified the roles of EIP-7910 and EIP-7951.
- Clarified that low-level `call{value: ...}("")` is available on `address` and is not exclusive to `address payable`; tightened the Solidity 0.8.37 custom-error compiler-bug note to its actual trigger conditions.
- Standardized all 16 article self-review checklists as unchecked learner progress items; stage/article publication status remains distinct from personal study progress.

## 2026-10-17

- Published the Storage Layout article for Stage 03, explaining slot and byte-offset assignments, inheritance, special state-variable categories, compiler layout output, custom storage bases, and upgrade compatibility.
- Added a dedicated Storage Layout overview diagram and linked the article from the Stage 03 index and canonical roadmap.
- Updated Stage 03 publication progress to 1/13; the individual learning checklist remains unchecked.

## 2026-10-17

- Published the Storage Slots article for Stage 03, explaining the slot-key/value distinction, 32-byte storage words, SLOAD/SSTORE, JSON-RPC inspection, account storage context, and common non-sequential locations.
- Added a dedicated Storage Slots diagram and linked the article from the Stage 03 index and canonical roadmap.
- Updated Stage 03 publication progress to 2/13; individual learning checkboxes remain unchecked.

## 2026-10-17

- Published the Storage Packing article for Stage 03, explaining Solidity packing rules, byte offsets, declaration-order examples, struct packing, gas trade-offs, and upgrade/assembly hazards.
- Added a dedicated Storage Packing overview diagram and linked the article from the Stage 03 index and canonical roadmap.
- Updated Stage 03 publication progress to 3/13; individual learning checkboxes remain unchecked.

## 2026-10-17

- Published the Mappings and Dynamic Arrays in Storage article for Stage 03, covering mapping key hashing, nested mappings, dynamic-array length/data slots, packed array elements, and the special storage encoding of bytes and string.
- Added a dedicated mapping/array storage diagram and linked the article from the Stage 03 index and canonical roadmap.
- Updated Stage 03 publication progress to 4/13; individual learning checkboxes remain unchecked.

## 2026-10-17

- Published the Memory Layout article for Stage 03, covering Solidity's reserved memory ranges, free memory pointer, zero slot, temporary object layouts, memory expansion gas, and memory-safe assembly conventions.
- Added a dedicated memory-map diagram and linked the article from the Stage 03 index and canonical roadmap.
- Updated Stage 03 publication progress to 5/13; individual learning checkboxes remain unchecked.

## 2026-10-17

- Published the Calldata Layout article for Stage 03, covering execution-frame input bytes, ABI selectors and head/tail layout, dynamic offsets, calldata opcodes, Solidity's `calldata` data location, gas considerations, and validation/security concerns.
- Added a dedicated calldata layout diagram and linked the article from the Stage 03 index and canonical roadmap.
- Updated Stage 03 publication progress to 6/13; individual learning checkboxes remain unchecked.

## 2026-10-17

- Published the Transient Storage (EIP-1153) article for Stage 03, covering TLOAD/TSTORE, transaction lifetime, execution-context ownership, revert semantics, static-call restrictions, Solidity value-type transient variables, and security considerations.
- Added a dedicated transient-storage diagram and linked the article from the Stage 03 index and canonical roadmap.
- Updated Stage 03 publication progress to 7/13; individual learning checkboxes remain unchecked.

## 2026-10-17

- Published the Storage Context article for Stage 03, explaining account-scoped storage, CALL/STATICCALL/DELEGATECALL context selection, proxy state, layout compatibility, initialization, and security considerations.
- Added a dedicated CALL-versus-DELEGATECALL storage-context diagram and linked the article from the Stage 03 index and canonical roadmap.
- Updated Stage 03 publication progress to 8/13; individual learning checkboxes remain unchecked.

## 2026-10-17

- Published the Storage Collisions article for Stage 03, explaining proxy metadata conflicts, incompatible upgrade layouts, Diamond facet collisions, custom slot collisions, and safe storage separation patterns.
- Added a dedicated collision-versus-namespaced-storage diagram and linked the article from the Stage 03 index and canonical roadmap.
- Included current references for EIP-1967, ERC-7201, ERC-8042, and ERC-2535.
- Updated Stage 03 publication progress to 9/13; individual learning checkboxes remain unchecked.


## 2026-10-17

- Published the Recursive Length Prefix (RLP) Encoding article, covering the five prefix cases, byte-string and list encoding, canonical integer conventions, decoding boundaries, Ethereum execution-layer usage, and security considerations.
- Added an RLP encoding overview diagram and linked the article from the Stage 03 checklist and canonical roadmap.
- Updated Stage 03 publication progress to 10/13; the concept checklist remains unchecked for study tracking.


## 2026-10-17

- Published the Merkle Proofs article, explaining inclusion and non-inclusion proofs, trusted roots, proof verification, Ethereum's Modified Merkle-Patricia Trie, the block header roots, and `eth_getProof`.
- Added a dedicated Merkle proof overview diagram and linked the article from the Stage 03 checklist and canonical roadmap.
- Updated Stage 03 publication progress to 11/13; the concept checklist remains unchecked for study tracking.


## 2026-10-17

- Published the Account Proofs article, covering the Ethereum account record, `keccak256(address)` trie-key derivation, verification against a block's `stateRoot`, `eth_getProof`, inclusion and non-inclusion, and the relationship to storage proofs.
- Added a dedicated account-proof verification diagram and linked the article from the Stage 03 checklist and canonical roadmap.
- Updated Stage 03 publication progress to 12/13; the concept checklist remains unchecked for study tracking.


## 2026-10-17

- Published the Storage Proofs article, explaining storage-slot derivation, the account `storageRoot`, MPT proof verification, `eth_getProof`, inclusion and non-inclusion, and how storage layout relates to the proven storage word.
- Added a dedicated storage-proof verification diagram and linked the article from the Stage 03 checklist and canonical roadmap.
- Completed Stage 03 — Storage and Data Layout with all 13/13 concept articles published. Individual checklist boxes remain unchecked for study tracking.


## 2026-10-17 — Stage 03 review

- Clarified that Solidity constructor arguments are ABI-encoded after deployment initcode, rather than using the regular external-function selector format.
- Fixed the inline complexity notation in the Merkle Proofs article and added a dated note linking the current MPT proof model to Ethereum's Verkle Trees roadmap.
- Added Verkle-roadmap context to the Account Proofs and Storage Proofs articles and adjusted those two specialist topics to Priority 2.
- Clarified that the Stage 03 checkboxes track personal study progress, while article publication status is recorded separately.


## 2026-10-17 — Stage 04 begins

- Published the Contract Composition article, including an interface-based fee-policy example, composition versus inheritance, dependency boundaries, and security/testing considerations.
- Added a dedicated Contract Composition architecture diagram and linked the article from the Stage 04 checklist and canonical roadmap.
- Updated Stage 04 publication progress to 1/17. The unchecked item remains a personal study checkbox.


## 2026-10-17 — Inheritance

- Published the Inheritance article covering visibility, `virtual`/`override`, `super`, base constructors, abstract contracts, multiple inheritance, C3 linearization, storage-layout implications, and security considerations.
- Added a dedicated inheritance overview diagram and linked the article from the Stage 04 study checklist and canonical roadmap.
- Updated Stage 04 publication progress to 2/17. The study checkbox remains unchecked until the concept is reviewed by the learner.


## 2026-10-17 — Interfaces and Libraries

- Published the Interfaces and Libraries article, covering interface declarations and runtime boundaries, ABI compatibility, ERC-165 discovery, internal versus public/external library functions, `using for`, library linking, and delegate-call context.
- Added a dedicated interface/library architecture diagram and linked the article from the Stage 04 checklist and canonical roadmap.
- Updated Stage 04 publication progress to 3/17. The study checkbox remains unchecked until the learner reviews the concept.


## 2026-10-17 — Internal and External Calls

- Published the Internal and External Calls article, covering internal function dispatch, external message-call frames, `this.function()`, call context, static calls, revert handling, `try/catch`, gas considerations, and reentrancy.
- Added a dedicated call-flow diagram and linked the article from the Stage 04 checklist and canonical roadmap.
- Updated Stage 04 publication progress to 4/17. The study checkbox remains unchecked until the learner reviews the concept.

## 2026-10-17

- Published the Proxy Contracts article, explaining proxy forwarding, DELEGATECALL execution context, implementation address resolution, initialization, upgradeability distinctions, and security concerns.
- Added a dedicated proxy architecture SVG diagram and linked the article from the Stage 04 index and canonical roadmap.
- Updated the root README and Stage 04 publication progress to 5/17; unchecked topic boxes remain personal study checklists.

## 2026-10-17 — Transparent Proxies

- Published the Transparent Proxies article, covering caller-based proxy routing, selector-clash mitigation, the OpenZeppelin Contracts 5.x `ProxyAdmin` relationship, the ERC-1967 upgrade path, and security considerations including UUPS interaction.
- Added a custom SVG diagram showing admin versus non-admin call routing and the `initialOwner` / `ProxyAdmin` distinction in OpenZeppelin 5.x.
- Linked the article from the Stage 04 index and full roadmap, and updated repository publication counters to 6/17. The study checkbox remains unchecked until the learner reviews the topic.

## 2026-10-17 — UUPS Proxies

- Published the UUPS Proxies article, covering implementation-resident upgrade logic, `_authorizeUpgrade`, `onlyProxy`, `proxiableUUID`, ERC-1967 implementation slots, initialization, storage compatibility, and security concerns.
- Added a custom SVG diagram showing delegated execution and the UUPS upgrade sequence.
- Linked the article from the Stage 04 index and canonical roadmap; updated repository publication progress to 7/17. The study checkbox remains unchecked until the learner reviews the topic.

## 2026-10-17 — Beacon Proxies

- Published the Beacon Proxies article, covering the proxy/beacon/implementation roles, ERC-1967 beacon slot, OpenZeppelin Contracts 5.x immutable beacon behavior, per-proxy initialization and storage, and fleet-wide upgrade/security trade-offs.
- Added a custom SVG architecture diagram showing multiple independent proxy storage contexts sharing an implementation through a single beacon and upgrade path.
- Linked the article from the Stage 04 index and canonical roadmap, and updated publication progress to 8/17. The study checkbox remains unchecked until the learner reviews the topic.

## 2026-10-17 — Minimal Proxies (Clones)

- Published the Minimal Proxies (Clones) article covering ERC-1167 forwarding bytecode, fixed implementation targets, clone-specific storage and initialization, OpenZeppelin `Clones`, CREATE2 address prediction, and security considerations.
- Added a dedicated SVG diagram showing one shared implementation serving multiple clones with independent storage, plus the deterministic deployment flow.
- Linked the article from the Stage 04 index and canonical roadmap, and updated publication progress to 9/17. The study checkbox remains unchecked until the learner reviews the topic.

## 2026-10-17 — Delegatecall-Based Proxy Architecture

- Published the Delegatecall-Based Proxy Architecture article covering CALL versus DELEGATECALL context, fallback dispatch, calldata and returndata forwarding, storage and event context, implementation resolution, and security boundaries.
- Added a custom SVG diagram showing the end-to-end forwarding lifecycle and proxy execution context.
- Linked the article from the Stage 04 index and canonical roadmap, and updated publication progress to 10/17. The study checkbox remains unchecked until the learner reviews the topic.

## 2026-10-17 — Contract Upgradeability

- Published the Contract Upgradeability article covering upgradeability trade-offs, Transparent/UUPS/Beacon/Diamond/clone patterns, upgrade lifecycle, storage compatibility, initialization, governance, validation, rollback limitations, and security considerations.
- Added a custom SVG diagram showing the proposed-change review and validation path, authorization, execution, post-upgrade verification, and monitoring.
- Linked the article from the Stage 04 index and canonical roadmap, and updated publication progress to 11/17. The study checkbox remains unchecked until the learner reviews the topic.

## 2026-10-17 — Storage Compatibility

- Published the Storage Compatibility article covering Solidity storage slot/offset rules, declaration and inheritance changes, structs and mappings, storage gaps, ERC-7201 namespaced storage, ERC-1967 metadata, and upgrade validation workflows.
- Added a custom SVG contrasting an unsafe inserted variable with a compatible append-only example and illustrating the storage-validation workflow.
- Linked the article from the Stage 04 index and canonical roadmap, and updated publication progress to 12/17. The study checkbox remains unchecked until the learner reviews the topic.

## 2026-10-17 — Initializers

- Published the Initializers article covering `initializer`, `reinitializer(version)`, `onlyInitializing`, `_disableInitializers()`, implementation locking, parent initializer order, atomic proxy setup, and versioned migrations.
- Added a custom SVG showing implementation locking, proxy initialization, and reinitialization during upgrades.
- Linked the article from the Stage 04 index and canonical roadmap, and updated publication progress to 13/17. The study checkbox remains unchecked until the learner reviews the topic.

## 2026-10-17 — Storage Collisions in Proxies

- Published the Storage Collisions in Proxies article distinguishing collisions between proxy metadata and implementation state, incompatible implementation-version layouts, and overlapping module/facet storage.
- Covered ERC-1967 unstructured metadata slots, storage-layout validation, namespaced storage, mapping and packed-slot risks, and realistic proxy testing.
- Added a dedicated SVG showing three collision types and their corresponding mitigation strategies.
- Linked the article from the Stage 04 index and canonical roadmap, and updated publication progress to 14/17. The study checkbox remains unchecked until the learner reviews the topic.

## 2026-10-17 — Diamond Pattern (EIP-2535)

- Published the Diamond Pattern article covering facets, selector-to-facet routing, `diamondCut` actions, `DiamondCut` events, loupe introspection, shared diamond storage, and security considerations.
- Added a dedicated SVG architecture diagram showing the single diamond address, multiple facets using `DELEGATECALL`, shared storage, and the cut/loupe interfaces.
- Linked the article from the Stage 04 index and canonical roadmap, and updated publication progress to 15/17. The study checkbox remains unchecked until the learner reviews the topic.

## 2026-10-17 — Deterministic Contract Deployment

- Published the Deterministic Contract Deployment article, distinguishing `CREATE` address derivation from `CREATE2` and explaining predictable addresses, factory patterns, cross-chain matching, failure cases, and authorization considerations.
- Added a dedicated SVG diagram for prediction, deployment, verification, and cross-chain address matching.
- Linked the article from the Stage 04 index and canonical roadmap, and updated publication progress to 16/17. The study checkbox remains unchecked until the learner reviews the topic.

## 2026-10-17 — CREATE2 Address Derivation

- Published the CREATE2 Address Derivation article with the exact EIP-1014 formula, 85-byte preimage layout, init-code hashing, Solidity prediction example, deployer-context rules, and common implementation mistakes.
- Added a dedicated SVG diagram illustrating the four preimage components, Keccak-256 hash, and extraction of the low 20 bytes.
- Linked the article from the Stage 04 index and canonical roadmap, and marked Stage 04 content complete at 17/17 published articles. Individual study checkboxes remain unchecked until reviewed.

## 2026-10-17 — Stage 04 audit and consistency pass

- Rechecked all 17 Stage 04 articles, their dedicated SVG references, and local Markdown links; no broken internal article or diagram paths were found.
- Clarified EIP-2535's requirement to include immutable function selectors as additions in the `DiamondCut` event and report them through loupe functions.
- Connected the Deterministic Contract Deployment and CREATE2 Address Derivation articles and made the full-roadmap status note consistently include Stage 03 completion.
- Source-code examples were reviewed editorially but were not compiled or executed as part of this pass.

## 2026-10-17 — Transaction Lifecycle

- Published the Transaction Lifecycle article covering transaction preparation, signing, submission, propagation, transaction pools, block inclusion, EVM execution, receipts, confirmations, and finality.
- Added a dedicated SVG diagram separating RPC submission, inclusion, execution success/revert, and finality.
- Linked the article from the Stage 05 index and canonical roadmap, and updated publication progress to 1/14 for Stage 05.
- The transaction lifecycle example is a JSON-RPC receipt request; no Solidity code was compiled or executed as part of this article.

## 2026-10-17 — Transaction Nonce

- Published the Transaction Nonce article covering sender transaction sequencing, nonce gaps, `latest` versus `pending` RPC views, same-nonce replacement behavior, included reverts, and the distinction between transaction nonces and EIP-7702 authorization nonces.
- Added a dedicated SVG diagram showing the account nonce sequence, a missing nonce blocking later transactions, and same-nonce replacement.
- Linked the article from the Stage 05 index and canonical roadmap, and updated Stage 05 publication progress to 2/14.

## 2026-10-17 — Transaction Types (EIP-2718)

- Published the Transaction Types article explaining the typed transaction envelope, legacy transaction encoding, type-specific payloads, signing-domain considerations, typed receipts, and major Ethereum transaction types.
- Covered type 1 (EIP-2930), type 2 (EIP-1559), type 3 (EIP-4844), and type 4 (EIP-7702), and explicitly distinguished a draft proposal for type 0x05 from active network support.
- Added a dedicated SVG diagram comparing the legacy format with the typed envelope and mapping major type identifiers to their defining features.
- Linked the article from the Stage 05 index and canonical roadmap, and updated Stage 05 publication progress to 3/14.

## 2026-10-17 — Access List Transactions (EIP-2930)

- Published the Access List Transactions article explaining the type-1 transaction payload, signed access-list structure, and relationship to EIP-2718 and EIP-2929.
- Documented access-list charges (2,400 gas per address and 1,900 per storage key), warm/cold accounting, potential gas trade-offs, duplicate entries, and why access lists are not permission lists.
- Added a dedicated SVG diagram showing the transaction envelope, access-list entries, upfront costs, and warm versus cold access behavior.
- Linked the article from the Stage 05 index and canonical roadmap, and updated Stage 05 publication progress to 4/14.
- The JSON-RPC request is an illustrative example and was not executed against a live Ethereum node.

## 2026-10-17 — Set Code Transactions (EIP-7702)

- Published the Set Code Transactions article explaining type 0x04, the outer transaction payload, authorization tuples and signatures, chain scoping, delegation indicators, and delegated execution context.
- Covered the distinction between outer transaction and authority nonces, authorization processing and skipped invalid tuples, persistent delegation and clearing, gas parameters, and security risks including mutable delegate code and unsafe `tx.origin` assumptions.
- Added a dedicated SVG diagram separating the outer transaction signature from authorization signatures and illustrating delegation resolution and its security boundary.
- Linked the article from the Stage 05 index and canonical roadmap, and updated Stage 05 publication progress to 5/14.
- No transaction was signed or broadcast, and no example was executed against a live network.

## 2026-10-17 — Chain ID and Replay Protection

- Published the Chain ID and Replay Protection article covering EIP-155-protected legacy transactions, typed transaction signing payloads, Chain ID versus network ID, and the distinction between chain scope and nonce-based replay prevention.
- Added developer guidance for EIP-712 domain separation, the CHAINID opcode / Solidity `block.chainid`, and EIP-7702 authorization tuples whose `chain_id` may be zero.
- Added a dedicated SVG diagram showing chain-specific signing, rejection on an alternate Chain ID, and separate defenses for same-chain transactions and off-chain signed messages.
- Linked the article from the Stage 05 index and canonical roadmap, and updated Stage 05 publication progress to 6/14.
- The signing payloads are conceptual examples; no transaction was signed, submitted, or executed.

## 2026-10-17 — Intrinsic Gas

- Published the Intrinsic Gas article distinguishing pre-execution transaction requirements from runtime EVM execution costs.
- Covered the base 21,000 gas charge, zero/non-zero calldata bytes, contract-creation overhead and EIP-3860 initcode word cost, EIP-2930 access-list charges, EIP-7702 authorization-list charges, and the EIP-7623 calldata floor.
- Added a dedicated SVG diagram illustrating the transaction-level cost components, calldata floor, and pre-execution gas-limit validation.
- Linked the article from the Stage 05 index and canonical roadmap, and updated Stage 05 publication progress to 7/14.
- Gas formulas are presented as educational summaries; no transaction was simulated or sent to a live node.

## 2026-10-17 — Gas Accounting

- Published the Gas Accounting article explaining EVM execution metering, intrinsic versus execution gas, final transaction gas accounting, refunds, revert/out-of-gas behavior, and EIP-1559 fee calculation.
- Covered memory expansion, state-dependent storage costs, warm/cold access pricing, the EIP-3529 refund cap, EIP-7623 calldata-floor interaction, and the separation between execution gas fees and blob fees.
- Added a dedicated SVG diagram showing the three layers of gas accounting: EVM work, final transaction gas usage, and ETH fee calculation.
- Linked the article from the Stage 05 index and canonical roadmap, and updated Stage 05 publication progress to 8/14.
- Gas examples are educational summaries; no code was profiled and no transaction was simulated or sent to a live node.

## 2026-10-17 — Gas Limit and Gas Used

- Published the Gas Limit and Gas Used article distinguishing the transaction's declared gas budget from final gas charged in the receipt, including `gasUsed` versus `cumulativeGasUsed`.
- Covered fee calculation, sender affordability checks, the distinction between transaction and block gas limits, success/revert/out-of-gas behavior, gas estimation caveats, and the EIP-7623 calldata floor.
- Added a dedicated SVG diagram comparing declared budget, actual gas used, unused gas, cumulative block usage, and block-level limits.
- Linked the article from the Stage 05 index and canonical roadmap, and updated Stage 05 publication progress to 9/14.
- Receipt values are illustrative; no live transaction was queried or submitted.

## 2026-10-17 — Out-of-Gas

- Published the Out-of-Gas article explaining exceptional halts, the difference between top-level and child-call gas exhaustion, and the distinct behavior of `REVERT` and insufficient intrinsic gas.
- Covered the EIP-150 63/64 gas-forwarding rule, state rollback and gas handling in nested calls, refund timing, top-level failed receipts, and practical debugging guidance.
- Added a dedicated SVG diagram contrasting top-level OOG, child-call OOG, normal REVERT, and validation failure due to insufficient intrinsic gas.
- Linked the article from the Stage 05 index and canonical roadmap, and updated Stage 05 publication progress to 10/14.
- Protocol behavior was researched against official EIP and Ethereum documentation; no transaction was executed on a live network.

## 2026-10-17 — Transaction Receipts

- Published the Transaction Receipts article covering the JSON-RPC receipt fields, execution status, gas usage, logs and bloom filters, contract creation results, typed receipt encoding, and inclusion versus finality.
- Distinguished the receipt's consensus payload from additional RPC metadata and documented `gasUsed` versus `cumulativeGasUsed`, blob-fee fields, pre-Byzantium root receipts, and receipt availability for pending transactions.
- Added a dedicated SVG diagram outlining how a processed transaction produces a receipt and which key information it exposes.
- Linked the article from the Stage 05 index and canonical roadmap, and updated Stage 05 publication progress to 11/14.
- JSON-RPC data is illustrative; no live transaction receipt was queried as part of the article's preparation.

## 2026-10-17 — Transaction Reverts

- Published the Transaction Reverts article explaining EVM `REVERT`, frame-scoped rollback, revert data, and Solidity failure patterns including `require`, explicit `revert`, custom errors, and `assert` / `Panic`.
- Covered the difference between caught inner-call failures and propagated top-level reverts, the rollback of state changes and logs, gas/nonce behavior, and why revert data must not be treated as proof of error origin.
- Added a dedicated SVG diagram contrasting caught versus propagated failures, revert payloads, and the distinct gas behavior of `REVERT` and out-of-gas.
- Linked the article from the Stage 05 index and canonical roadmap, and updated Stage 05 publication progress to 12/14.
- Solidity examples are explanatory snippets and were not compiled or executed as part of this article's preparation.

## 2026-10-17 — Transaction Replacement

- Published the Transaction Replacement article explaining same-sender/same-nonce replacement for pending transactions and why replacement is governed by local txpool policy rather than a universal consensus rule.
- Covered fee-field requirements for legacy/type-1, EIP-1559 type-2, and blob transactions; speed-up versus cancel; and why cancellation is a race rather than a guaranteed deletion.
- Added a dedicated SVG diagram showing the original and replacement competing for the same nonce, txpool price-bump checks, and canonical inclusion consequences.
- Linked the article from the Stage 05 index and canonical roadmap, and updated Stage 05 publication progress to 13/14.
- Replacement behavior is explained from official client policy documentation; no transaction was broadcast or tested on a live node.

## 2026-10-17 — Gas Refunds

- Published the Gas Refunds article explaining the EIP-2200 storage refund counter and how original, current, and new slot values affect net refund accounting.
- Covered EIP-3529's one-fifth refund cap, the 4,800-gas storage-clear schedule under the referenced EIP-2929 / EIP-2930 parameters, and removal of the `SELFDESTRUCT` refund.
- Documented frame-level rollback of refund adjustments, why refunds are applied after execution, and the interaction with the EIP-7623 calldata floor and final fee calculation.
- Added a dedicated SVG diagram showing execution-time counter adjustments and end-of-transaction settlement.
- Linked the article from the Stage 05 index and canonical roadmap; Stage 05 is now content-complete at 14/14 published articles.
- Examples are explanatory only; no contract was compiled, and no live transaction or gas profiler was used.

## 2026-10-17 — Stage 05 Technical Review

- Audited the 14 Stage 05 concept articles and verified that all local Markdown references and linked SVG diagrams resolve to files in the repository.
- Corrected the hexadecimal `cumulativeGasUsed` example in Gas Limit and Gas Used and Transaction Receipts so `0x1dc90` matches 122,000 gas.
- Clarified the EIP-7623 calldata-floor formula in Gas Refunds and distinguished the still-proposed EIP-7976 parameter change from active protocol rules.
- Removed a duplicated sentence from the Stage 05 index. Stage 05 remains content-complete at 14/14 published articles.

## 2026-10-17 — JSON-RPC

- Published the Stage 06 JSON-RPC article, introducing JSON-RPC request/response structure and its separation from Ethereum's Execution API method definitions.
- Covered request IDs, result versus error responses, standard JSON-RPC error codes, Ethereum `QUANTITY` versus `DATA` encoding, block identifiers, and HTTP / WebSocket / IPC transports.
- Added practical `curl` and JSON examples, batch-request and notification caveats, and security guidance for endpoint exposure, credentials, provider trust, retries, and transaction submission.
- Added a custom SVG diagram showing an application sending a JSON-RPC request through a transport to an Ethereum execution client and receiving a result or error.
- Linked the article from the Stage 06 index and canonical roadmap, and updated Stage 06 publication progress to 1/17.
- Examples are illustrative; the placeholder endpoint was not contacted and no live RPC request or transaction was submitted.

## 2026-10-17 — Stage 06 Application Connectivity Complete

- Published the remaining 16 Stage 06 concept articles: Ethereum RPC Methods, Reading Blockchain State, Sending Transactions, Transaction Signing, `eth_call`, `eth_estimateGas`, `eth_sendRawTransaction`, Log and Event Filtering, WebSocket Subscriptions, Execution Clients, Archive Nodes, Full Nodes, Light Clients, Execution API, Consensus API, and Engine API.
- Added a dedicated SVG diagram for each new concept and linked the articles throughout the Stage 06 index and canonical roadmap.
- Covered standard Ethereum RPC method families, read and simulation semantics, transaction signing/submission workflows, log querying and reorg handling, node storage modes, light-client verification, and the separation between Execution, Consensus/Beacon, and Engine APIs.
- Updated the Stage 06 index, roadmap status, and root README to show all 17/17 articles published.
- Examples are educational illustrations; no live RPC endpoint or transaction was used during article preparation.

## 2026-10-17 — Stage 06 Technical Review

- Reviewed all 17 Stage 06 articles and checked their local Markdown and SVG references against the repository tree.
- Removed two stale duplicate diagram references in Sending Transactions and Execution Clients; each of the 17 articles now has one valid dedicated SVG reference.
- Added concept-specific developer review checklists and key takeaways where missing, and aligned the numbered section headings.
- Clarified Beacon API content negotiation: JSON is the default, while SSZ request/response support is available only on endpoints that specify it.
- Confirmed Stage 06 publication status remains complete at 17/17; no live RPC calls or transactions were performed as part of this review.


## 2026-10-18 — Stage 07: Fraud Proofs

- Published the Fraud Proofs concept article with a custom SVG diagram, developer-focused examples, security considerations, and references to Ethereum and OP Stack specifications.
- Linked the article from the Stage 07 index and canonical roadmap; updated the root README and roadmap overview to show Stage 07 progress at 1/19 articles.
- Kept the remaining 18 concept files and checklist entries unchanged.
