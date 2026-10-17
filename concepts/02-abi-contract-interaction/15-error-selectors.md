# Error Selectors in Solidity and the EVM

> **Stage:** 02 — ABI and Contract Interaction  
> **Priority:** 1  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-17

## 1. Overview

An **error selector** is a four-byte identifier at the beginning of the ABI-encoded data for a Solidity error. It is derived from the first four bytes of the Keccak-256 hash of the error's canonical signature.

For example, consider a custom error:

    error InsufficientBalance(uint256 available, uint256 required);

Its canonical signature is:

    InsufficientBalance(uint256,uint256)

The selector for this signature is <code>0xcf479181</code>. When this error is reverted with arguments, the revert payload starts with that four-byte selector, followed by the ABI-encoded values of <code>available</code> and <code>required</code>.

![Diagram showing a custom error signature being canonicalized and hashed with Keccak-256, taking the first four bytes as the error selector, then prepending the selector to ABI-encoded arguments in revert data](https://raw.githubusercontent.com/0xSix9/Blockchain-Concepts-Learning-Roadmap/main/assets/diagrams/error-selectors-flow.svg?sanitize=true)

[Open the Error Selectors diagram directly](https://github.com/0xSix9/Blockchain-Concepts-Learning-Roadmap/blob/main/assets/diagrams/error-selectors-flow.svg)

An error selector resembles a function selector in how it is derived, but it serves a different purpose. A function selector usually identifies a function in a call's calldata. An error selector identifies the error format at the start of revert data.

## 2. How an Error Selector Is Derived

To understand an error selector, follow these steps:

1. Start with the error name and its parameter types.
2. Write the **canonical signature**: the name followed by the comma-separated canonical parameter types, with no spaces.
3. Compute the Keccak-256 hash of that signature.
4. Take the first four bytes (eight hexadecimal characters after the <code>0x</code> prefix).

For example:

    error InsufficientBalance(uint256 available, uint256 required);

The canonical signature is:

    InsufficientBalance(uint256,uint256)

The parameter names <code>available</code> and <code>required</code> are not part of the signature. Neither are source-file paths, contract names, return types, or data-location specifiers. The parameter types and their order do matter.

### Use Keccak-256, not the standardized SHA3-256 function

Ethereum uses **Keccak-256** for selectors. Although Keccak and the later standardized SHA3-256 algorithm are closely related, their padding differs, so their hashes are not interchangeable. If you compute a selector with a generic SHA3-256 function instead of Ethereum-compatible Keccak-256, you can get the wrong result.

In practice, use an Ethereum-aware library or the selector exposed by the compiled ABI tooling instead of implementing the hash function yourself.

### Reading the selector in Solidity

Solidity exposes the selector of a declared error through its `selector` member:

    bytes4 selector = InsufficientBalance.selector;

The expression gives the same four-byte selector that appears at the beginning of that error's revert payload. It is useful in tests, low-level error inspection, and code that needs to compare a selector explicitly. It does not decode the error arguments or prove that a particular contract produced the revert data.

## 3. Selector Versus Complete Revert Data

The error selector is only the first four bytes. It is not the entire error and does not contain the argument values.

For this declaration:

    error InsufficientBalance(uint256 available, uint256 required);

A revert such as:

    revert InsufficientBalance(120, 200);

has a payload with this conceptual layout:

    0xcf479181
    ++ ABI-encoded uint256(120)
    ++ ABI-encoded uint256(200)

Each static <code>uint256</code> argument is encoded in a 32-byte ABI word. The example's selector identifies the error layout; the following words provide the values.

A client decodes the revert payload by matching the selector against an expected error ABI and decoding the bytes that follow it. Because ABI data is not self-describing, a selector alone does not tell a client every parameter type, argument name, or explanation. The client needs the corresponding ABI or another trusted description of the error.

### Parameterless errors

A parameterless error still has a selector:

    error Unauthorized();

Its revert payload contains the four-byte selector and no encoded argument words. Tooling may have a friendly error name available from the ABI, but that name is not transmitted as plain text in the payload.

## 4. Error Selectors Versus Function Selectors

Both selectors use the first four bytes of the Keccak-256 hash of a canonical signature. The main difference is **where each selector appears and how it is used**.

| Property | Function selector | Error selector |
|---|---|---|
| Appears in | Usually transaction or message-call calldata | Revert data returned by failed execution |
| Identifies | A function signature | An error signature |
| Derived from | Canonical function signature | Canonical error signature |
| Bytes used | First 4 bytes of Keccak-256 hash | First 4 bytes of Keccak-256 hash |
| Followed by | ABI-encoded function arguments | ABI-encoded error arguments |
| Typical consumer | Function dispatcher, client, ABI encoder | Client, debugger, caller, revert-data decoder |

For example, the calldata for a call to <code>transfer(address,uint256)</code> starts with the function selector for that function, followed by its encoded arguments. If execution then reverts with <code>InsufficientBalance(uint256,uint256)</code>, the returned revert data begins with the error selector, followed by that error's encoded arguments.

Do not confuse the selector in the original calldata with the selector in the revert response. They identify separate interface entries.

## 5. Built-In Error Selectors: Error and Panic

Solidity uses built-in error types for common revert paths.

### Error(string)

A revert string such as:

    revert("Insufficient balance");

is encoded using the built-in error format <code>Error(string)</code>. Its selector is:

    0x08c379a0

The selector is followed by ABI-encoded string data (including the normal ABI offset, length, and string bytes). A client can decode this format when it recognizes <code>Error(string)</code>.

### Panic(uint256)

The built-in error type <code>Panic(uint256)</code> is used for certain language-defined or compiler-inserted failures, including checked arithmetic overflow/underflow and some invalid operations. Its selector is:

    0x4e487b71

The value following the selector is a panic code that helps identify the category of failure. It is not a custom error declared by your contract.

Some representative panic codes include:

- <code>0x01</code> — failed <code>assert</code>.
- <code>0x11</code> — arithmetic overflow or underflow in checked arithmetic.
- <code>0x12</code> — division or modulo by zero.
- <code>0x32</code> — out-of-bounds access in certain array or bytes operations.
- <code>0x41</code> — memory allocation too large or out of memory.

Use official Solidity documentation for the complete set and the precise situations that trigger each panic. A panic often points to an invariant violation, invalid operation, or bug rather than a normal business-rule rejection.

### Custom errors

A custom error uses the same general layout, but its selector comes from the signature that you define. For example:

    error Unauthorized(address caller);

This gives a typed, machine-decodable failure that can include context. For additional detail on declarations, encoding, compiler versions, and error handling, see [Custom Errors](14-custom-errors.md).

## 6. Selectors Are Short and Can Collide

An error selector is only 32 bits long. Two different canonical signatures can theoretically produce the same first four bytes of their Keccak-256 hashes. This is called a **selector collision**.

That has practical consequences:

- A selector is not a globally unique identity for an error.
- A decoder should know the expected ABI and execution context.
- If an interface contains multiple errors with the same selector, the four-byte prefix alone cannot uniquely determine which error was intended.
- The selector does not prove which contract produced the error data.

The ABI specification reserves the selectors <code>0x00000000</code> and <code>0xffffffff</code> for future use. When designing custom errors, avoid relying on these reserved selector values.

### Why signature canonicalization matters

A mistake in the signature string changes the hash and therefore changes the selector. Common mistakes include:

- Including spaces in the canonical signature.
- Including parameter names, such as writing <code>InsufficientBalance(uint256 available,uint256 required)</code> instead of <code>InsufficientBalance(uint256,uint256)</code>.
- Using the wrong canonical type spelling, such as hashing <code>uint</code> rather than its canonical form <code>uint256</code>.
- Reordering parameters.
- Hashing with SHA3-256 rather than Ethereum Keccak-256.

When debugging a mismatch, first inspect the exact canonical signature and compiler ABI; then verify the hashing implementation.

## 7. Decoding Error Selectors in Tools and Contracts

### Off-chain decoding

JavaScript libraries such as ethers or viem can use an ABI to parse revert data. A typical workflow is:

1. Get the raw revert data from a simulation, RPC error response, transaction trace, or a caught low-level call.
2. Read the first four bytes.
3. Match that selector against the errors known in the ABI.
4. Decode the remaining bytes according to that error's parameter types.
5. Show a user-friendly message while retaining the raw data for debugging.

The exact error-data surface depends on the RPC method, node, wallet, and client library. Some interfaces expose raw revert data; others wrap it inside a structured error object. On-chain transaction receipts normally record the failed status, but do not reliably carry the complete revert payload as a standard receipt field.

### On-chain handling

Solidity does not provide a special statically typed <code>catch SomeCustomError(...)</code> clause. A contract can catch a failed external call using <code>try/catch</code> or inspect raw return data from a low-level call. Selector checks may be part of such handling, but should be designed carefully.

A contract should not blindly trust any error merely because the first four bytes match a known selector. A nested callee can revert with its own error, and revert data bubbles up through external calls by default unless an intermediate contract catches or transforms it. A malicious callee can also fabricate bytes that mimic a trusted contract's error format.

### Error data is not an authorization signal

Do not grant permissions, mark an operation successful, or make a security-critical decision solely because returned data begins with a particular error selector. Validate the relevant caller, target, state, and success/failure result separately.

## 8. Practical Example: Inspecting an Error

Suppose a contract declares:

    error SlippageTooHigh(uint256 actualOutput, uint256 minimumOutput);

and executes:

    revert SlippageTooHigh(95, 100);

A developer can reason about the result as follows:

- The canonical signature is <code>SlippageTooHigh(uint256,uint256)</code>.
- The first four bytes of the Keccak-256 hash of that signature identify the error layout.
- The next two ABI words represent <code>actualOutput = 95</code> and <code>minimumOutput = 100</code>.
- The caller can decode those values if it has the matching error ABI.
- The payload does not prove that a particular trusted contract originated the error; a nested or malicious contract can return matching bytes.

For protocols with many errors, keep error definitions in interfaces or a shared library where appropriate, maintain ABI artifacts for integrations, and test decoding for representative revert paths.

## 9. Common Mistakes and Security Considerations

1. **Treating the error selector as the full revert payload.** Arguments follow the selector and may contain essential context.
2. **Using the wrong hash algorithm.** Use Ethereum Keccak-256, not standardized SHA3-256.
3. **Hashing a non-canonical signature.** The signature uses the error name and canonical parameter types, not parameter names or spaces.
4. **Assuming the selector is globally unique.** Four-byte selectors can collide.
5. **Confusing built-in errors with custom errors.** <code>Error(string)</code> and <code>Panic(uint256)</code> have standardized selectors and distinct payload layouts.
6. **Trusting the selector as provenance.** Revert data can bubble up from external calls and can be deliberately forged.
7. **Misreading RPC error responses.** Wallets and RPC providers can wrap revert data differently; extract and validate the raw bytes before decoding.
8. **Ignoring reserved selector values.** <code>0x00000000</code> and <code>0xffffffff</code> are reserved by the ABI specification.
9. **Assuming a failed transaction receipt contains the full error.** The receipt usually records transaction status and logs; raw revert data may need simulation, tracing, or an RPC error response.
10. **Using revert data for critical control flow without validation.** Treat it as diagnostic data, not proof of authorization or identity.

## 10. Key Takeaways

- An error selector is the first four bytes of the Keccak-256 hash of the error's canonical signature.
- Revert data consists of the selector followed by ABI-encoded error arguments.
- Error selectors are derived similarly to function selectors, but appear in revert data rather than the original calldata.
- <code>Error(string)</code> uses selector <code>0x08c379a0</code>; <code>Panic(uint256)</code> uses <code>0x4e487b71</code>.
- Selectors are only four bytes and can collide; the ABI and call context are needed to decode reliably.
- The ABI specification reserves <code>0x00000000</code> and <code>0xffffffff</code>.
- Error data can bubble up from nested calls or be forged, so it is not a trustworthy proof of origin or authorization.

## References

- [Solidity Documentation — ABI Specification: Errors](https://docs.solidity.org/en/latest/abi-spec.html#errors)
- [Solidity Documentation — Contracts: Custom Errors](https://docs.solidity.org/en/latest/contracts.html#custom-errors)
- [Solidity Documentation — Error Handling and Panic Codes](https://docs.solidity.org/en/latest/control-structures.html#panic-via-assert-and-error-via-require)
- [Custom Errors](14-custom-errors.md)
- [Return Data and Revert Data](../01-evm-execution/10-return-data-and-revert-data.md)
- [Function Selectors](03-function-selectors.md)

## Review Checklist

- [ ] Derive an error selector from its canonical signature using Keccak-256.
- [ ] Distinguish an error selector from a function selector and from a complete revert payload.
- [ ] Recognize the built-in `Error(string)` and `Panic(uint256)` selectors.
- [ ] Explain reserved selectors, selector collisions, revert bubbling, and spoofed error data.
- [ ] Describe how a client decodes revert data with the expected ABI.
- [ ] Consult the official Solidity ABI and error-handling documentation for exact behavior.
