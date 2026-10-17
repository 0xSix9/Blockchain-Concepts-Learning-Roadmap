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
