# Forced Inclusion: Bypassing a Censoring Sequencer

> **Stage:** 07 — Advanced Layer 2 and Cross-Chain  
> **Priority:** 3  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-18

## 1. Overview

Forced inclusion is a censorship-resistance mechanism that lets a user submit a transaction or message through a path other than the ordinary sequencer path—often by publishing it to an L1 contract or queue. The rollup protocol then requires the sequencer or the state-transition process to account for that input under specified conditions.

![Forced Inclusion: Bypassing a Censoring Sequencer](../../assets/diagrams/08-forced-inclusion.svg)

## 2. Why It Matters to a Developer

A sequencer can be available and fast while still refusing to include a user's transaction. Forced inclusion addresses this failure mode and is a practical test of whether a rollup has a meaningful escape hatch from its normal operator.

## 3. How It Works

1. The user sends a transaction or deposit message to an L1 contract or an on-chain queue.
2. The protocol records the message and its ordering/timing metadata.
3. A deadline or protocol rule determines when the queued item must be included or when an alternative derivation path takes over.
4. L2 derivation or sequencing rules include the forced transaction in a later L2 block, or reject it only under protocol-defined conditions.
5. Independent L2 nodes derive a consistent chain from the available L1 inputs and the rollup's rules.

The queue format, timeout, fee payment, transaction encoding, and exact inclusion semantics depend on the rollup. Some protocols make L1 deposits part of the L2 input stream, which is not identical to every generic forced-transaction design.

## 4. Example or Experiment

An L2 sequencer ignores a valid user transaction. Where supported, the user can submit the required forced input to L1 and wait for the protocol's deadline. The user should check whether the route requires a special L1 contract call, a deposit-format transaction, extra gas, or a longer inclusion delay before treating it as a reliable fallback.

## 5. Common Mistakes and Security Considerations

Forced inclusion is not necessarily immediate and may be expensive during L1 congestion. A forced input can still be constrained by protocol validity rules; it cannot force the chain to accept a transaction that violates nonce, balance, signature, or execution constraints. Confirm who can submit it, what events prove queue inclusion, whether a sequencer can delay it, and whether the chain can continue deriving blocks during an outage.

## 6. Key Takeaways

- Forced inclusion is an alternate path to resist sequencer censorship.
- The exact queue and timeout are protocol-specific, and the transaction must still satisfy validity rules.
- A credible design documents a permissionless route and what happens if the normal sequencer stops.

## 7. References

- [Ethereum.org — Optimistic Rollups](https://ethereum.org/developers/docs/scaling/optimistic-rollups/)
- [Optimism Specs — Deposits](https://specs.optimism.io/protocol/deposits.html)
- [Ethereum.org — Zero-Knowledge Rollups](https://ethereum.org/developers/docs/scaling/zk-rollups/)
