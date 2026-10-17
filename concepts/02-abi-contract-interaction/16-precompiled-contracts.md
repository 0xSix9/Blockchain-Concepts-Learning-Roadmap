# Precompiled Contracts in the EVM

> **Stage:** 02 — ABI and Contract Interaction  
> **Priority:** 1  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-17

## 1. Overview

A **precompiled contract** is a protocol-defined operation exposed at a special address and executed by an Ethereum client using a dedicated implementation rather than ordinary deployed EVM runtime bytecode. Precompiles make selected operations—such as hashing, elliptic-curve calculations, and cryptographic proof checks—available to contracts with standardized input formats and gas costs.

They are called *contracts* because EVM code interacts with them through contract-like addresses and call mechanisms. They are *precompiled* because the operation is implemented by the execution client instead of being interpreted as normal EVM bytecode stored at that address.

Precompiles matter because some cryptographic operations would be extremely expensive to implement from ordinary EVM instructions. A protocol-level implementation can execute them more efficiently while keeping the result and gas accounting consistent across conforming clients.

![Diagram showing a smart contract calling a reserved precompile address, the execution client performing a protocol-defined operation, and output bytes returning to the caller](https://raw.githubusercontent.com/0xSix9/Blockchain-Concepts-Learning-Roadmap/main/assets/diagrams/precompiled-contracts-overview.svg?sanitize=true)

[Open the Precompiled Contracts diagram directly](https://github.com/0xSix9/Blockchain-Concepts-Learning-Roadmap/blob/main/assets/diagrams/precompiled-contracts-overview.svg)

## 2. What Makes a Precompile Different?

A normal Solidity contract has deployed runtime bytecode. The EVM executes that bytecode when a message call targets its address.

A precompile uses a different execution path:

- Its address is assigned by the applicable protocol rules.
- The execution client recognizes that address and runs the corresponding built-in implementation.
- The call supplies bytes as input and receives bytes as output, or a failure.
- Its required gas is defined by protocol specifications and can depend on the input.
- Every conforming execution client must produce compatible behavior, because different results could cause nodes to disagree about the state transition.

A precompile usually has no ordinary Solidity source or deployed runtime bytecode that explains its implementation. It is still accessed through EVM calls, however, so a Solidity contract can interact with it through a high-level built-in such as `sha256(data)` or through a low-level call to its address.

**Important distinction:** the precompile address and its behavior are part of the chain's execution rules; they are not created by deploying your own Solidity contract to that address.

## 3. Common Ethereum Precompile Addresses

The following table describes the standard precompile set represented by the post-Prague execution configuration. Address assignments and gas schedules are fork-dependent, so check the target chain's activated rules before relying on any entry.

| Address | Common name | Main operation |
|---|---|---|
| `0x01` | ECRECOVER | Recover an Ethereum-style secp256k1 signer address from signature data |
| `0x02` | SHA-256 | Compute a SHA-256 hash |
| `0x03` | RIPEMD-160 | Compute a RIPEMD-160 hash; result is returned in a 32-byte word |
| `0x04` | Identity | Return a copy of the input bytes |
| `0x05` | Modular exponentiation (MODEXP) | Compute modular exponentiation, used by some cryptographic schemes |
| `0x06` | BN254 addition | Add points on the BN254 (also called alt_bn128) curve |
| `0x07` | BN254 scalar multiplication | Multiply a BN254 curve point by a scalar |
| `0x08` | BN254 pairing check | Perform pairing checks used by certain proof systems |
| `0x09` | BLAKE2F | Run the BLAKE2b compression function with the specified parameters |
| `0x0a` | KZG point evaluation | Verify a KZG proof for a blob-related point evaluation, as specified by EIP-4844 |
| `0x0b` | BLS12-381 G1 addition | Add points in the G1 group |
| `0x0c` | BLS12-381 G1 MSM | Perform G1 multi-scalar multiplication |
| `0x0d` | BLS12-381 G2 addition | Add points in the G2 group |
| `0x0e` | BLS12-381 G2 MSM | Perform G2 multi-scalar multiplication |
| `0x0f` | BLS12-381 pairing check | Evaluate a pairing equation across input pairs |
| `0x10` | BLS12-381 map to G1 | Map a base-field element to a G1 curve point |
| `0x11` | BLS12-381 map to G2 | Map an extension-field element to a G2 curve point |

The first entries include general-purpose hash and signature operations; the later entries support elliptic-curve arithmetic, pairings, and KZG/BLS-related cryptography. A smart-contract developer does not need to reimplement these algorithms in Solidity to use them, but should understand what the operation verifies and what its input bytes mean.

### Additional precompiles can be fork-specific

The list above is not a promise that every EVM-compatible chain has exactly the same precompile set. For example, the Osaka configuration described by EIP-7910 adds `P256VERIFY` at `0x0100` for secp256r1/P-256 signature verification (EIP-7951). Its availability depends on the target chain and the fork it has activated. Never assume that an address is a precompile just because it is one on another network.

## 4. How a Call to a Precompile Works

At a simplified level, a contract interaction follows this sequence:

1. The caller prepares the input bytes required by that precompile's specification.
2. The caller executes a message call, commonly using `STATICCALL` when it only needs a result and no state change.
3. The EVM recognizes the destination address as an active precompile under the current chain rules.
4. The execution client validates and processes the input using its dedicated implementation, charging the specified gas.
5. The operation returns output bytes, or reports failure.
6. The caller checks the call result and validates or decodes the output.

The EVM does not hash a Solidity function name or decode a general JSON ABI on behalf of a precompile. Each precompile has its own specified byte layout. Some data resembles ABI words, but a low-level call must use the exact input format defined for that operation.

That difference is useful when thinking about the preceding concepts in this stage: normal contract methods commonly use a four-byte function selector followed by ABI-encoded arguments; a direct precompile call generally provides the operation's prescribed input bytes, not a regular Solidity function selector.

## 5. Example: Calling the SHA-256 Precompile

For routine hashing, Solidity already provides the built-in `sha256` function. This example shows the underlying call pattern explicitly so that the mechanism is visible:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract Sha256PrecompileExample {
    function hash(bytes calldata data) external view returns (bytes32) {
        (bool success, bytes memory output) =
            address(0x02).staticcall(data);

        require(success, "SHA-256 precompile call failed");
        require(output.length == 32, "Unexpected output length");

        return abi.decode(output, (bytes32));
    }
}
```

What happens here:

- `address(0x02)` is the SHA-256 precompile address.
- `staticcall(data)` sends the raw bytes in `data`; it does not prepend a Solidity function selector.
- `success` tells the caller whether the EVM call succeeded.
- The expected result is exactly 32 bytes, the length of a SHA-256 digest.
- The returned bytes are decoded as `bytes32`.

In production code, prefer Solidity's `sha256(data)` built-in unless you have a reason to perform the low-level call yourself. The example is educational: it demonstrates that a precompile can be reached using the same low-level call mechanism that contracts use for other destinations.

Also note the length check. A call to an address with no applicable code or precompile can sometimes succeed while returning empty data; success alone does not establish that the expected operation ran. Output validation is therefore important when a contract relies on an optional or chain-specific precompile.

## 6. Why Precompiles Matter to Smart-Contract Developers

### Cryptographic verification

Applications may need signature recovery or verification, elliptic-curve calculations, or pairing operations. Precompiles provide specified implementations that would generally be costly to reproduce using ordinary EVM instructions.

### Proof systems and data availability

The BN254 and BLS12-381 operations are relevant to systems that use elliptic-curve cryptography and pairings. The KZG point-evaluation precompile is tied to Ethereum's blob-related data-availability mechanisms. Using one of these precompiles does not itself prove that an application has designed its overall verification logic correctly; the caller must still supply the right inputs and enforce the right conditions.

### Gas estimation and debugging

Precompile gas is not necessarily a single constant. Some operations charge based on input length or complexity, and protocol upgrades can change gas schedules. Incorrect input encoding, insufficient gas, or an unexpected return format can make a call fail. When estimating or debugging a transaction, inspect the precompile's current specification and test the exact call path.

### Cross-chain and fork compatibility

Two EVM-compatible chains may expose different precompiles, assign special behavior to different addresses, or use different gas schedules. Code that depends on a precompile should document the required chain and fork assumptions and should not infer availability from an address alone.

## 7. Failure Behavior and Security Considerations

A precompile is a low-level primitive, not an application-level security guarantee. Pay attention to the following:

- **Validate success and output.** Check whether the call succeeded, then verify output length and value according to the relevant specification.
- **Understand failure semantics.** Invalid input and out-of-gas behavior are defined per precompile. Do not assume that every precompile consumes gas or reports failure identically.
- **Use correct input encoding.** Elliptic-curve and proof operations have strict byte layouts and validation rules. A malformed input may fail or produce a result that the caller handles incorrectly.
- **Do not confuse computation with authorization.** Recovering a signer address or verifying a cryptographic proof does not automatically authorize a user. The surrounding contract must compare the result against the expected signer, commitment, domain, message, or policy.
- **Treat gas schedules as versioned protocol rules.** A gas assumption made for one fork may not hold after an upgrade.
- **Check precompile availability on the actual target.** A missing precompile may behave like an address without code for some call patterns. Do not silently treat empty output as a valid cryptographic result.

These checks are especially important in signature validation, bridge verification, account abstraction, and proof-verification contracts, where a single incorrect assumption can undermine the application's security boundary.

## 8. Precompiles vs. Opcodes, Normal Contracts, and System Contracts

| Mechanism | What it is | How it is executed |
|---|---|---|
| EVM opcode | An instruction in EVM bytecode, such as `KECCAK256` or `STATICCALL` | The EVM interprets the instruction as part of the current execution |
| Normal contract | An account with deployed runtime bytecode | The EVM executes the code at the destination address |
| Precompiled contract | A protocol-defined operation mapped to a special address | The execution client runs the corresponding implementation when the address is called |
| System contract | A protocol- or ecosystem-defined contract that typically has ordinary contract code at a designated address and may be invoked through specific system mechanisms | Depends on its deployment, code, and the relevant protocol rules |

The categories are related but not interchangeable. For example, `KECCAK256` is an opcode, while SHA-256 is available through a precompile. A precompile is not simply a Solidity library, and a normal contract does not become a precompile because its address is well known.

## 9. Practical Checklist

Before relying on a precompile in an application, ask:

- Is the precompile active on the exact chain and fork where the contract will run?
- Does the input match the official specification byte-for-byte?
- Is the expected output length and meaning checked?
- Does the contract handle call failure and malformed or unexpected output safely?
- Does the gas estimate reflect the relevant fork and the actual input?
- Is the cryptographic result being used for the correct purpose, with all required authorization and domain checks?

## 10. References

- [Ethereum Execution Layer Specifications](https://github.com/ethereum/execution-specs) — reference specification and fork-specific execution behavior.
- [EIP-7910: `eth_config` JSON-RPC method](https://eips.ethereum.org/EIPS/eip-7910) — includes fork-specific enumerations of precompile sets.
- [EIP-196: Precompiled contracts for addition and scalar multiplication on the elliptic curve alt_bn128](https://eips.ethereum.org/EIPS/eip-196).
- [EIP-197: Pairing checks for the elliptic curve alt_bn128](https://eips.ethereum.org/EIPS/eip-197).
- [EIP-152: BLAKE2 compression function `F` precompile](https://eips.ethereum.org/EIPS/eip-152).
- [EIP-2565: ModExp gas cost](https://eips.ethereum.org/EIPS/eip-2565).
- [EIP-4844: Shard Blob Transactions](https://eips.ethereum.org/EIPS/eip-4844) — defines the KZG point-evaluation precompile.
- [EIP-2537: Precompile for BLS12-381 curve operations](https://eips.ethereum.org/EIPS/eip-2537).
- [EIP-7951: Precompile for secp256r1 curve support](https://eips.ethereum.org/EIPS/eip-7951).
- [Solidity documentation: built-in functions](https://docs.solidity.org/en/latest/units-and-global-variables.html#mathematical-and-cryptographic-functions) — see `ecrecover`, `sha256`, and `ripemd160`.

---

**Key takeaway:** a precompiled contract exposes an efficient, protocol-defined operation at a special address. To use it safely, understand its exact input/output format, gas behavior, and availability on the target chain and fork.
