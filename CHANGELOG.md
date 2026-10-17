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
