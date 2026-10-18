# Rollup Bridges: Moving Assets and Messages Between L1 and L2

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

![Rollup bridge architecture: paired L1 and L2 bridge contracts communicate through an authenticated message system. Deposits lock or escrow assets on L1 and create corresponding L2 balances; withdrawals burn or lock L2 assets, then release the L1 asset after the required proof, challenge, and finality conditions.](../../assets/diagrams/10-rollup-bridges.svg)

## 1. Overview

A **rollup bridge** is a set of contracts and message-processing rules that moves assets or information between a rollup and its parent chain. In an Ethereum rollup, the bridge usually connects Ethereum Layer 1 (L1) with a Layer 2 (L2) network. It may transfer ETH or ERC-20 tokens, relay arbitrary messages, or enable a smart contract on one layer to call a counterpart on the other.

A canonical rollup bridge is not merely a website or token-wrapping interface. It includes the onchain contracts that hold or release assets, the message protocol that authenticates cross-layer instructions, and the verification rules that determine when a withdrawal can be completed.

Rollup bridges commonly rely on the rollup's own security architecture. For a rollup that uses Ethereum for data availability and settlement, an L2-to-L1 message may need to be published, associated with a recognized output commitment, and proven before the L1 contract executes it. Optimistic rollups generally require the relevant challenge conditions to be satisfied; validity-proof rollups use their protocol's proof-verification and finality rules. Exact mechanics depend on the rollup.

## 2. Why a bridge needs paired contracts and authenticated messages

Ethereum and an L2 are separate execution environments with distinct state. A contract on L1 cannot directly read an L2 contract's storage, and an L2 contract cannot directly transfer an L1 token. Each layer needs a protocol for communicating a claim or instruction to the other.

A common design uses paired contracts:

- **L1 bridge or gateway:** holds, escrows, or releases the source-chain asset and initiates messages to L2.
- **L2 bridge or gateway:** mints, releases, burns, or locks the corresponding L2 representation and initiates messages back to L1.
- **Cross-domain messenger or rollup message system:** transports messages and establishes their origin, destination, and authenticated sender.
- **Settlement and proof contracts:** provide the conditions under which child-to-parent messages may be accepted by the parent chain.

Paired contracts must not trust arbitrary calls that merely claim to be from the other layer. The receiving side needs to authenticate the protocol messenger and the original counterpart bridge. Otherwise, an attacker could call a minting function directly and create unbacked tokens, or forge a withdrawal message and attempt to release escrowed assets.

The bridge's security therefore depends on the full message path: the message must be valid for the source chain, originate from the authorized contract, be authenticated by the bridge protocol, and execute only under the destination bridge's expected rules.

## 3. The two basic directions

In common OP Stack terminology, a **deposit** is L1 → L2 and a **withdrawal** is L2 → L1. Some systems use different interface wording, so always check the target bridge's documentation.

### Deposit: L1 → L2

For a typical asset that originates on Ethereum, the L1 bridge takes custody of the asset and sends an authenticated message to the L2 bridge. The L2 side then credits the user with a corresponding representation—often by minting a bridge-specific token.

A simplified ERC-20 deposit works like this:

1. The user approves the L1 bridge to transfer the ERC-20 token, unless the token's deposit method does not need an approval.
2. The user calls the L1 bridge's deposit function and specifies the amount, destination, and relevant gas or message parameters.
3. The L1 bridge transfers the token into escrow and initiates a cross-domain message.
4. The protocol carries that message to the L2 execution environment.
5. The L2 bridge authenticates that the message came through the trusted messenger and from the expected L1 bridge.
6. The L2 gateway checks that the destination token is the registered or valid counterpart for the L1 asset, then mints or credits the corresponding L2 representation.

The L1 transaction can succeed before the L2-side message has executed. That separation is intentional: an L1 deposit creates a cross-chain instruction that must still be processed on the child chain. Rollups may have retryable tickets, gas limits, or message-retry flows for deposits that do not complete on the first attempt.

### Withdrawal: L2 → L1

A withdrawal commonly reverses the economic operation, but it is not just the deposit process running backwards. A user's L2 representation must be removed from circulation or locked, and the L1 bridge must receive an authenticated instruction before releasing the L1 asset.

A simplified withdrawal works like this:

1. The user calls the L2 bridge or token gateway.
2. The L2-side representation is burned or otherwise accounted for according to the bridge's token model.
3. The L2 bridge sends a message to the L1 bridge through the rollup's cross-domain messaging mechanism.
4. The withdrawal message becomes part of the rollup's canonical L2 history and is tied to a recognized state or output commitment.
5. The user or a relayer supplies the required withdrawal proof or claim transaction on L1.
6. The rollup's protocol-specific conditions are checked, including any challenge, proof-maturity, or finality requirements.
7. The L1 bridge authenticates the message and releases the escrowed asset to the designated recipient.
8. The bridge marks the message as executed so it cannot be replayed.

For optimistic rollups, a standard withdrawal can involve a waiting period related to the fault-proof or challenge process. That delay is part of the security model, not simply a slow token transfer. Some third-party bridges provide faster liquidity, but their early payout changes who carries the delay and counterparty risk; it does not remove the canonical bridge's protocol requirements.

## 4. How token representations work

The token shown in a user's wallet on L2 is not necessarily the same token contract as the L1 asset. A bridge needs an explicit mapping between the two.

### Lock and mint

A common rollup model holds the original L1 token in an L1 escrow contract and mints a corresponding L2 representation when the deposit message is processed. On withdrawal, the L2 representation is burned, and the L1 contract releases the escrowed asset after verifying the withdrawal message.

This design aims to keep the L2 representation backed by the L1 asset held by the canonical bridge, subject to the token's behavior and the bridge implementation's assumptions.

### Burn and mint

Some systems burn a token on the source layer and mint an equivalent token on the destination layer. This requires minting authority, protocol verification, and clear supply accounting. It can be suitable for a token native to the system, but it changes the trust model compared with simply escrowing a canonical L1 token.

### Native tokens and bridged tokens with the same ticker

A ticker or display name does not establish token identity. An L2 may have a bridged representation of an L1 USDC token and a distinct token that is native to the L2 or issued by another entity. They may share a ticker while having different issuer, backing, redemption, and trust properties.

Applications should identify assets using the chain ID and contract address, and verify the registered L1/L2 mapping. Never assume that two assets are equivalent because they have the same symbol, name, decimals, or icon.

## 5. Canonical bridge versus third-party bridge

### Canonical rollup bridge

A **canonical bridge** is the bridge designed to work with the rollup's native contracts and security model. Deposits and withdrawals are authenticated through the rollup's official message path, and the bridge uses the chain's expected output, proof, and settlement rules. On many Ethereum rollups this is the preferred baseline for an L1 ↔ L2 transfer, though developers still need to inspect upgrade authorities, contract versions, and token mappings.

For example, OP Stack's standard bridges operate on top of cross-domain messenger contracts. The L1 bridge escrows an L1 token and sends a message to the L2 bridge, which mints its mapped representation. For withdrawals, the L2-side token is burned and the authenticated message eventually causes the L1 bridge to unlock the escrowed asset.

Arbitrum's canonical token bridge uses paired bridge and gateway contracts. Parent-to-child deposits use the Inbox and related message mechanisms, while child-to-parent withdrawals use the L2-to-L1 messaging and Outbox flow after the applicable challenge conditions. The exact components and flow depend on the child chain and supported gateway type.

### Third-party or liquidity bridge

A third-party bridge may use liquidity pools, validators, relayers, a committee, or a separate messaging network to accelerate or route an asset transfer. It may give the user an asset on the destination chain before the canonical rollup withdrawal has completed.

This can improve speed and convenience, but its guarantees differ. The user may rely on liquidity providers, an external verification network, token inventory, or a bridge-specific contract. A fast transfer is not necessarily a canonical L1 withdrawal. Risk analysis should identify the party that pays immediately, the source of destination liquidity, how the provider later recovers funds, and what happens if the canonical message is delayed or fails.

Do not categorize bridges purely by interface or branding. Inspect the mechanism that authenticates the source-chain event and authorizes the destination-chain payment.

## 6. Cross-domain messaging is part of the bridge

Modern rollup bridges often transfer messages rather than tokens directly. A message can trigger a function on the other chain, and a token transfer is one use of this capability.

The message system must bind the message to the correct domains. Important fields or implied context include:

- Source chain and destination chain.
- Source sender and destination target.
- Message payload and value.
- A unique identifier, nonce, or message hash.
- The proof, messenger state, or settlement condition that authenticates delivery.
- An execution status that prevents replay.

This is why a receiving bridge commonly checks both that the call came from its trusted cross-domain messenger and that the original sender on the source chain was the paired bridge contract. Checking only the immediate caller is not always sufficient; the messenger's authenticated source-sender information matters too.

A destination message can also fail after successful submission. Some protocols support retries, while others require a new transaction or expose a specific failure recovery path. The bridge user interface should distinguish message initiation from destination execution, not just report the first transaction as a completed transfer.

## 7. Security model: what protects the assets?

A rollup bridge is only as secure as the contracts and protocol rules that authorize a transfer. Several components must work together.

### Source-chain escrow and token accounting

For lock-and-mint bridges, the contract must transfer and account for the actual source-chain asset correctly. The accounting should prevent minting more destination tokens than the accepted source amount, apart from explicitly documented token behaviors or fees. Token hooks, rebasing, fee-on-transfer behavior, or non-standard ERC-20 implementations can violate simplistic assumptions.

### Authenticated cross-domain sender

A bridge must reject a message if it did not originate from the paired bridge through the canonical messenger. Otherwise, an arbitrary caller could mint destination representations, release tokens from escrow, or invoke privileged bridge logic.

### Recognized state commitments and withdrawal proofs

For L2 → L1 withdrawals, the L1 bridge cannot simply trust a user-supplied message. The message must be included in the rollup's recognized output or message commitment, and the protocol must enforce the appropriate proof or dispute conditions. A valid Merkle inclusion proof against an unrecognized root proves very little.

### Challenge and finality rules

Optimistic rollups may require the message to pass through a challenge period before execution on L1. The client should use the bridge contract's real eligibility checks and documented protocol state rather than calculating a local timer and assuming that the withdrawal is ready.

### Replay protection

A message must not execute twice. Contracts should mark a message as consumed as part of the successful execution path and include sufficient domain separation to stop an otherwise similar message on another chain or contract from being replayed.

### Upgrade keys and emergency controls

Canonical bridge contracts can be upgradeable. That may be necessary for protocol upgrades, but administrators, security councils, proxy admins, or guardians may have the power to change the bridge's behavior. This authority is part of the security model. Review who can upgrade or pause the bridge, whether changes are timelocked, and how the system handles a compromised administrator.

## 8. Common bridge security failures

### Forged message or incorrect origin check

If the destination bridge fails to authenticate the messenger and source bridge, a malicious contract may submit a fake deposit-finalization or withdrawal-finalization call.

### Incorrect token mapping

If an application maps the wrong L1 token to an L2 representation, it may mint an asset that is not backed by the intended source token. The mapping registry, gateway router, decimals, and counterpart token interface should all be checked.

### Double execution or replay

If a message can be executed more than once, a single withdrawal can release escrowed assets repeatedly. State updates that mark a message consumed should occur atomically with the successful withdrawal and should be keyed to the correct message domain.

### Reentrancy and unsafe token assumptions

Transfers to arbitrary token contracts or recipients can trigger callbacks. State changes and external calls must be ordered safely, and the bridge should define how it handles non-standard token behavior, fee-on-transfer assets, and failed transfers.

### Confusing soft confirmation with a completed withdrawal

A withdrawal transaction appearing on an L2 explorer does not mean the L1 asset is already released. The canonical message may still require output publication, proof submission, challenge completion, and L1 execution. Interfaces that collapse these stages can mislead users and downstream applications.

### Excessive trust in a fast or externally verified bridge

A third-party bridge may depend on a validator set, guardian quorum, liquidity provider, relayer network, or contract upgrade authority. If that infrastructure is compromised, destination assets may be released without the same guarantee as a canonical rollup message. The user should know which component is actually verifying the source event.

## 9. Developer checklist

Before integrating a rollup bridge, verify:

- Which exact L1 and L2 bridge/gateway contracts are official for the chain?
- What chain IDs and token contract addresses define the source asset and destination representation?
- Does the bridge lock, burn, mint, release, or otherwise account for the asset? Is the token mapping canonical?
- Which messenger authenticates messages, and how does the destination verify the original source sender?
- What state root, message commitment, proof, challenge condition, and finality rule authorize withdrawals?
- How does the bridge prevent message replay, duplicate withdrawal, and cross-domain confusion?
- Can messages fail after initiation, and how are retries or refunds performed?
- Which contracts are upgradeable or pausable, and who controls those roles?
- Is the bridge canonical or third-party? If third-party, which validators, liquidity sources, relayers, or guardians must be trusted?
- Does the UI distinguish deposit submitted, L2 message executed, withdrawal initiated, withdrawal claimable, and withdrawal finalized?
- What happens if the token has non-standard transfer behavior or its supply changes outside the bridge?
- Has the full flow been tested against the actual deployed contract addresses and the current protocol version?

## 10. Key takeaways

- A rollup bridge connects L1 and L2 through paired contracts and authenticated cross-domain messages.
- Deposits and withdrawals are different execution paths with different escrow, token-mapping, proof, and finality requirements.
- A canonical bridge follows the rollup's native messaging and settlement design; a third-party or liquidity bridge can have a different trust model.
- A successful L2 withdrawal transaction is not the same as an L1 withdrawal that has been proven, finalized, and executed.
- Token symbols do not prove asset equivalence. Verify chain IDs, contract addresses, and the official mapping.
- Bridge security depends on source-asset accounting, correct sender authentication, accepted commitments, proof verification, replay protection, and upgrade governance.

## References

1. [Ethereum.org — Bridges](https://ethereum.org/developers/docs/bridges/)
2. [OP Stack Specification — Standard Bridges](https://specs.optimism.io/protocol/bridges.html)
3. [OP Stack Specification — Predeploys and L2StandardBridge](https://specs.optimism.io/protocol/predeploys.html)
4. [Ethereum.org — Optimism Standard Bridge Contract Walkthrough](https://ethereum.org/developers/tutorials/optimism-std-bridge-annotated-code/)
5. [Arbitrum Docs — Bridging Overview](https://docs.arbitrum.io/arbitrum-essentials/bridging/overview)
6. [Arbitrum Docs — Token Bridging](https://developer.arbitrum.io/how-arbitrum-works/deep-dives/token-bridging)
7. [Arbitrum Docs — Parent-to-Child Chain Messaging](https://docs.arbitrum.io/how-arbitrum-works/deep-dives/l1-to-l2-messaging)
8. [Arbitrum Docs — Child-to-Parent Chain Messaging](https://docs.arbitrum.io/how-arbitrum-works/deep-dives/l2-to-l1-messaging)
