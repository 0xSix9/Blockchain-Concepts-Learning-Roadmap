# Forced Withdrawals: Exiting an L2 When the Operator Fails

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

![Forced withdrawal escape hatch: a user requests withdrawal through L1, the operator must service it or prove it invalid, and an ignored request can trigger a freeze-and-escape path using a state proof.](../../assets/diagrams/09-forced-withdrawals.svg)

## 1. Overview

A **forced withdrawal** is an escape or recovery mechanism intended to let a user move assets out of a Layer 2 (L2) system without relying entirely on the normal operator-controlled transaction path. The exact meaning varies across protocols. Some systems let a user place a withdrawal request directly on Layer 1 (L1), require the operator to execute it or prove it invalid within a grace period, and allow the application to be frozen if the request is ignored. Other designs aim to let users withdraw directly against a frozen, authenticated state using a Merkle proof.

This mechanism addresses an important availability and self-custody problem: even if a rollup or application prevents invalid state transitions, a centralized sequencer, state proposer, or offchain operator can still refuse to process a user's valid request—or stop operating altogether. A valid state can protect the correctness of balances, but users also need a way to access those balances when the ordinary withdrawal flow breaks down.

**Forced withdrawal is not a universal protocol feature with one standard implementation.** Depending on the design, it may be an application-level forced request, an escape hatch after a state-update timeout, or a direct exit path from a frozen state. Do not assume that every L2 exposes a forced-withdrawal function just because it supports normal withdrawals or forced inclusion.

## 2. Why normal withdrawal may not be enough

A routine L2-to-L1 withdrawal generally begins with a transaction executed on L2. The withdrawal is recorded in the L2 state, a state or output commitment is made available on L1, and the user or a relayer supplies a proof to the L1 bridge. The withdrawal can then be finalized once the protocol's required proof, challenge, maturity, and finality conditions have been satisfied.

This path assumes the user can get the L2 withdrawal transaction processed and that the protocol can publish or accept the relevant state commitment. Failure of either dependency can strand assets even when the user controls their keys.

Typical failure cases include:

- **Sequencer censorship:** the sequencer refuses to include a transaction that would start the user's withdrawal.
- **State-proposer failure:** no new state root, output, or validity proof is submitted to L1, so the normal exit proof cannot be completed.
- **Operator outage:** an offchain service stops processing requests, and the application has no normal route to update the L2 state.
- **Application-level censorship:** the operator refuses an action even though the user can submit a request on L1.
- **Prolonged liveness failure:** state updates stop for long enough that the protocol needs an emergency way to preserve withdrawals from a known state.

A forced-withdrawal mechanism tries to close one or more of these gaps. It must specify which failure it is designed to handle; a way to bypass a sequencer does not automatically solve a failure of the state proposer or data-availability system.

## 3. Forced withdrawal, forced inclusion, and normal withdrawal

These concepts are closely related, but they provide different guarantees.

### Normal withdrawal

A user performs the protocol's ordinary withdrawal transaction on L2. The withdrawal becomes part of an accepted state or output; the user proves inclusion through the bridge's recognized commitment and finalizes it under the bridge's rules. This is the normal exit path, and it may require waiting for a challenge period or other security conditions.

### Forced inclusion

A user submits an eligible transaction or message through an alternate path, often an L1 inbox. If the sequencer fails to include it within the permitted window, the protocol's fallback rules can force it into the L2 input stream. This is primarily a way to bypass a censoring or unavailable sequencer. It does not necessarily provide a separate way to exit when new state commitments can no longer be produced.

### Forced withdrawal / escape hatch

A user directly requests an exit through an L1 contract or another protocol-enforced path, without relying solely on the normal L2 operator workflow. Depending on the design, the operator must service or challenge that request, or the system eventually enters a frozen escape mode in which users prove their balances against a recognized state root and withdraw through L1.

| Mechanism | Primary purpose | Typical dependency |
|---|---|---|
| Normal withdrawal | Exit through the ordinary bridge workflow | L2 execution, output commitment, proof/finality rules |
| Forced inclusion | Bypass sequencer censorship or outage | L1 inbox or mandatory input-derivation rule |
| Forced withdrawal | Exit when normal operator processing fails | L1 request, timeout/escape rules, authenticated state and withdrawal proof |

The table is conceptual. Individual protocols may combine these mechanisms, use different names, or provide only some of them.

## 4. Common forced-withdrawal patterns

### Pattern A — Forced request followed by operator service

The user sends a withdrawal request directly to an L1 contract. The contract records the request onchain and starts a grace period. The operator must process the request or provide protocol-defined evidence that the request is invalid.

This is a useful anti-censorship design because the operator cannot silently discard a valid request without consequences. The protocol must define what counts as a valid request, which user or vault is authorized, how the request is matched to an offchain operation, and how the contract proves that an invalid request need not be served.

### Pattern B — Freeze and escape

If the operator ignores a valid request until the grace period expires, the protocol can allow the application or exchange contract to be frozen. A frozen application cannot keep publishing ordinary state updates, but the escape mechanism remains available.

Users then prove that their account, vault, or position was included in the last recognized state. Once the proof is verified, the protocol moves the proven balance into an onchain withdrawal path. This preserves safety by using an authenticated state rather than accepting a user-supplied balance.

This design has a serious consequence: freezing can stop all users from making progress in the application, not just the censored user. It is an emergency escape path rather than a frictionless substitute for normal operation.

### Pattern C — Idle proposer or emergency state progression

Some escape-hatch designs target a failed state proposer. If no valid state update is produced for a defined period, a fallback may let another actor propose or resolve the state, or may freeze state progression and let users withdraw from the last accepted commitment.

This addresses a different liveness dependency than ordinary sequencer censorship. A forced transaction is useful only if someone can still incorporate it and advance the canonical state. If no valid output or proof can be produced, the protocol may need a stronger mechanism that does not depend on a new L2 block being processed.

These patterns can be combined, but each needs separate reasoning about validity, liveness, and custody.

## 5. Example: StarkEx Spot escape hatch

StarkEx Spot documentation illustrates a forced full-withdrawal design with an explicit freeze-and-escape path. This is an example of an application-specific escape hatch, not a universal implementation for all rollups.

At a high level, the documented flow is:

1. The user calls **fullWithdrawalRequest** on L1, identifying the vault and the key or owner information required by the contract.
2. The operator has a configured **freeze grace period** to process the request or prove that it is invalid.
3. If the request remains pending after the grace period, **freezeRequest** can freeze the application contract. Under the documented rules, anyone may be able to trigger the freeze when the valid request has remained unserved.
4. The user obtains a Merkle authentication path for their vault against the frozen, recognized vault-tree root.
5. The user submits the proof to the escape verifier, then calls the escape function for the verified vault and balance.
6. The assets become available through the protocol's onchain withdrawal mechanism, with replay-protection checks preventing the same vault balance from being escaped more than once.

The current StarkEx Spot documentation describes a default **FREEZE_GRACE_PERIOD** of seven days for this flow. That is a property of the documented StarkEx configuration, not a general rule for every L2. The contract and version deployed by a particular application are authoritative.

StarkEx Perpetual applications have related forced operations but different position semantics. For example, a user may need to close a synthetic position through a forced trade before withdrawing collateral. A design that works for a simple token balance cannot be assumed to work identically for leveraged positions, lending balances, LP shares, or assets whose value depends on application logic.

## 6. A generic freeze-and-escape lifecycle

A protocol that implements a withdrawal escape hatch needs a precisely specified sequence.

1. **Initiate the request on L1.** The contract records the request, the affected account or position, and the relevant identifiers.
2. **Allow normal processing.** The operator is given a defined period to process the request or prove it invalid, depending on the design.
3. **Check the grace period and request status.** A freeze should not be triggered merely because time passed if the request was already served or validly rejected.
4. **Freeze state progression when required.** The protocol prevents new state updates from invalidating the state root used for escape. The freeze transition must be unambiguous and enforceable onchain.
5. **Identify the frozen state.** Users need access to the recognized state root and the data required to derive their balances or positions.
6. **Verify an account or vault proof.** The escape verifier checks the Merkle path against the frozen root, along with the correct key, asset identifier, vault, amount, and other domain-specific fields.
7. **Register the escaped balance.** The contract marks the escape as used and credits an onchain withdrawal balance or equivalent claim.
8. **Withdraw to L1.** The owner performs the final withdrawal, and the contract prevents double withdrawal or duplicate proof use.

The details vary widely. Some systems use an escape request rather than freezing the entire application; some allow a new proposer to restore liveness; some require the user to produce a proof; others expose proof-building tools. The guarantee is only as strong as the deployed state-transition and escape contracts.

## 7. What a forced withdrawal does—and does not—guarantee

### It can reduce dependence on the operator

A well-designed mechanism lets a user start an exit from L1 and continue even if the ordinary operator workflow is unavailable. After the defined condition is met, the operator should not be able to veto every valid withdrawal indefinitely.

### It is not necessarily immediate

The user may need to wait for the request grace period, a freeze timeout, L1 transaction inclusion, proof verification, and normal withdrawal processing. This is an emergency liveness guarantee, not an instant exit guarantee.

### It does not prove an arbitrary balance

The contract must verify the balance against a recognized state commitment. A self-reported amount or an unauthenticated Merkle root is insufficient. The protocol needs a trusted commitment path, a correct proof verifier, and a clearly defined way to bind the leaf to the user's identity and asset.

### It cannot withdraw value that the state does not assign to the user

If assets are locked in a lending market, used as collateral, represented by a complex LP position, or subject to application-specific debt and accounting, a simple balance proof may not be enough to return the user's economic value safely. The application must define whether escape recovers only liquid vault balances or also handles positions and liabilities.

### It depends on data availability

To prove a balance in a frozen state, users need the state data or enough published information to construct a valid proof. If relevant data was withheld and cannot be reconstructed, a state root by itself does not reveal the user's balance. Data availability and escape-hatch usability must be considered together.

### It may freeze the application

A system that freezes the whole application in response to an unserved withdrawal request can protect users from indefinite censorship, but it also disrupts users who were not attempting to exit. The freeze conditions need to be resistant to malicious requests, and the operator must not be able to evade them by submitting irrelevant or invalid responses.

## 8. Security risks and design trade-offs

### Incorrect state root or stale state

The escape mechanism must be tied to a canonical state root that the settlement layer recognizes. If it accepts a root chosen by the operator at escape time, the operator could publish or substitute a state that omits balances. The contract should define when the root becomes frozen and how it is authenticated.

### Malformed Merkle proofs and leaf ambiguity

The proof verifier must validate the path, tree height, leaf encoding, key derivation, asset identifier, and claimed amount. If two different logical accounts can map to the same leaf representation or if important fields are omitted from the commitment, a proof may be interpreted incorrectly.

### Replay and double-withdrawal

An escape should not be executable twice for the same vault, asset, or balance. The contract should consume the relevant escape claim before transferring funds and ensure that a repeated proof cannot restore an already withdrawn balance.

### Denial-of-service requests

If forced-withdrawal requests are cheap, an attacker might flood the contract with requests and make the application freeze too easily or increase operator burden. StarkEx documentation describes a premium-cost mechanism for certain forced operations as one defense. Protocols may use fees, rate limits, validation, or per-account constraints, but these controls should not make honest users unable to exit.

### Governance and pausing powers

An administrator may be able to pause withdrawals, alter trusted state proposers, change escape parameters, or upgrade the verifier. These powers can be useful for emergency response but weaken unconditional escape guarantees. Review how the escape path behaves if the administrator is malicious, unavailable, or compromised.

### Incomplete escape for composable positions

An account may hold an ERC-20 balance, but it may also own an application position whose value depends on open orders, debt, liquidation rules, or offchain matching. A generic Merkle proof does not automatically unwind these relationships. Protocol designers must state exactly which assets and positions can be exited and what losses or settlement assumptions remain.

## 9. Developer and protocol review checklist

When reviewing a forced-withdrawal mechanism, ask:

- What exact failure does it address: sequencer censorship, state-proposer failure, application censorship, or a full operator outage?
- Which L1 contract records the forced request, and how are the user, vault, asset, and request uniquely identified?
- What is the grace period in the deployed configuration, and what event starts it?
- Must the operator serve the request, prove it invalid, or submit an alternative valid state update?
- Who can freeze the application or activate escape mode, and what prevents a malicious user from freezing it with an invalid request?
- Which state root is used for escape, and how is that root authenticated and frozen?
- What data must users obtain to construct a Merkle proof? Is it published, archived, and realistically accessible during an outage?
- Does the proof bind the balance to the correct owner, asset, vault, and chain?
- How does the protocol prevent duplicate escape, replay, and double withdrawal?
- Can the mechanism recover tokens only, or also positions, collateral, LP shares, debts, and pending application claims?
- Which privileged roles can pause, upgrade, or disable the mechanism?
- Has the escape procedure been tested end-to-end with the operator offline, the normal UI unavailable, and no cooperation from the application team?

## 10. Key takeaways

- Forced withdrawals are escape mechanisms for exiting an L2 or app-specific rollup when the normal operator-controlled withdrawal path fails.
- They are different from forced inclusion: one aims to include a transaction, the other aims to preserve a user's exit path.
- Some implementations require the operator to service or reject a request within a grace period; an ignored request can enable a freeze-and-escape process.
- Escape proofs must be checked against an authenticated state root and bound to the correct owner, asset, and account data.
- A state root without the data needed to construct an inclusion proof does not guarantee usable self-custody.
- Forced withdrawal is not universal, instant, or automatically compatible with complex DeFi positions; the deployed contract's behavior and trust assumptions must be checked.

## References

1. [StarkEx Spot — Withdrawing and Escaping Without Application Approval](https://docs.starkware.co/starkex/spot/withdrawing_and_escaping_without_app_approval.html)
2. [StarkEx Spot — Forced Operations Overview](https://docs.starkware.co/starkex/spot/shared/forced-operations_overview.html)
3. [StarkEx Perpetual — Performing Forced Transactions](https://docs.starkware.co/starkex/perpetual/performing-forced-transactions.html)
4. [OP Stack Specification — Withdrawals](https://specs.optimism.io/protocol/withdrawals.html)
5. [OP Stack Specification — L2 Output Root Proposals](https://specs.optimism.io/protocol/proposals.html)
6. [Ethereum.org — Zero-Knowledge Rollups](https://ethereum.org/developers/docs/scaling/zk-rollups/)
7. [Figueira et al. — A Practical Rollup Escape Hatch Design](https://arxiv.org/abs/2503.23986)
8. [Quantstamp — L2 Security Framework: Escape Hatches](https://github.com/quantstamp/l2-security-framework)
