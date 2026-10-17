# Custom Errors in Solidity: Structured and Efficient Reverts

> **Stage:** 02 — ABI and Contract Interaction  
> **Priority:** 1  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-17

## 1. Overview

A **custom error** is a Solidity-defined error type that describes why an operation failed and may include useful data about the failure. Instead of reverting with a descriptive string, a contract can define a named error and use it with <code>revert</code>.

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.24;

    error InsufficientBalance(uint256 available, uint256 required);

    contract SimpleVault {
        mapping(address => uint256) public balance;

        function withdraw(uint256 amount) external {
            uint256 available = balance[msg.sender];

            if (amount > available) {
                revert InsufficientBalance(available, amount);
            }

            balance[msg.sender] = available - amount;
        }
    }

The error name explains the failure category, and the arguments provide structured details. Solidity encodes the error into revert data that can be returned to an external caller, shown by developer tooling, or inspected by an off-chain application.

![Diagram showing a failed Solidity check creating custom-error revert data, with a four-byte error selector followed by ABI-encoded arguments, then being returned through external calls for decoding by a client](https://raw.githubusercontent.com/0xSix9/Blockchain-Concepts-Learning-Roadmap/main/assets/diagrams/custom-errors-revert-data.svg?sanitize=true)

[Open the Custom Errors diagram directly](https://github.com/0xSix9/Blockchain-Concepts-Learning-Roadmap/blob/main/assets/diagrams/custom-errors-revert-data.svg)

Custom errors were introduced in Solidity 0.8.4. They can be declared at file scope or inside contracts, interfaces, and libraries. The declarations describe errors; the actual revert happens only when code executes a corresponding <code>revert</code> statement or a supported <code>require</code> call fails.

## 2. Why Use Custom Errors?

Traditionally, contracts often used revert strings:

    require(amount <= available, "Insufficient balance");

Custom errors provide several advantages:

- **Structured meaning:** an error has a declared name and typed parameters.
- **Useful context:** callers can inspect values such as the available balance and requested amount without parsing natural-language strings.
- **Gas and bytecode efficiency:** custom errors often reduce deployment bytecode and the cost of constructing revert information compared with long strings, although exact savings depend on the contract, arguments, and compiler optimizations.
- **Consistent interfaces:** a protocol can define an error set in an interface so clients and integrating contracts can decode common failures.

An error without parameters needs only its four-byte selector in the revert payload. When parameters are present, their ABI-encoded values follow that selector.

Custom errors do not make a failed operation costless. A failing transaction still consumes gas for the work performed before the revert, and the error's parameters also need to be encoded when the failure occurs.

## 3. Declaring and Reverting With a Custom Error

The syntax resembles a function declaration but uses the <code>error</code> keyword:

    error Unauthorized();
    error InvalidAmount(uint256 provided, uint256 minimum);

To use the error, construct it in a <code>revert</code> statement:

    if (msg.sender != owner) {
        revert Unauthorized();
    }

    if (amount < minimum) {
        revert InvalidAmount(amount, minimum);
    }

When the condition fails, Solidity stops the current execution path, reverts state changes made in the reverted call frame, and returns the error data. If the failure propagates through the external call chain, callers receive the revert data unless an intermediate contract catches and handles it.

### A complete example

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.24;

    error Unauthorized(address caller);
    error InvalidAmount(uint256 provided, uint256 minimum);
    error InsufficientBalance(uint256 available, uint256 required);

    contract WithdrawalVault {
        address public immutable owner;
        uint256 public immutable minimumWithdrawal;
        mapping(address => uint256) public credits;

        constructor(uint256 minimumWithdrawal_) {
            owner = msg.sender;
            minimumWithdrawal = minimumWithdrawal_;
        }

        function withdraw(uint256 amount) external {
            if (amount < minimumWithdrawal) {
                revert InvalidAmount(amount, minimumWithdrawal);
            }

            uint256 available = credits[msg.sender];
            if (amount > available) {
                revert InsufficientBalance(available, amount);
            }

            credits[msg.sender] = available - amount;
        }

        function setCredit(address user, uint256 amount) external {
            if (msg.sender != owner) {
                revert Unauthorized(msg.sender);
            }
            credits[user] = amount;
        }
    }

Each error describes a distinct failure condition. The caller of <code>setCredit</code> can distinguish an authorization failure from an amount-related failure without trying to parse English sentences.

The example focuses on error handling; it does not send Ether during withdrawal. Production withdrawal logic must separately address outgoing-transfer success, reentrancy, and accounting invariants.

### Defining errors in interfaces

Errors may also be declared in interfaces to make the expected failure vocabulary part of an integration's documented interface:

    interface IVault {
        error Unauthorized(address caller);
        error InsufficientBalance(uint256 available, uint256 required);

        function withdraw(uint256 amount) external;
    }

A contract can refer to an interface error using its qualified name, such as <code>revert IVault.InsufficientBalance(available, amount)</code>. Clients can use the same ABI error definition to decode returned revert data.

Defining the same error signature in separate scopes does not give the resulting data a unique source identity. On the wire, the error name and parameter types determine the selector; the source file or contract where an identical error was declared is not encoded as provenance.

## 4. Custom Errors vs. Revert Strings

These approaches communicate a failure differently:

    require(amount <= available, "Insufficient balance");

    if (amount > available) {
        revert InsufficientBalance(available, amount);
    }

The first uses the built-in <code>Error(string)</code> format, whose selector is <code>0x08c379a0</code>, followed by ABI-encoded string data. The second uses the custom error signature and its encoded arguments.

| Property | Revert string | Custom error |
|---|---|---|
| Failure description | Free-form text | Named, typed error |
| Parameters | Usually included in a string or assembled text | Explicit typed arguments |
| Client handling | Display or parse the message string | Decode by selector and ABI |
| Bytecode and gas | Often larger for long strings | Often more efficient, depending on usage |
| Stability | Wording may change | Signature can serve as a more stable interface if kept compatible |

Custom errors are especially useful when a failure benefits from typed data. A short generic failure string can still be appropriate in some contexts; use the representation that best serves the contract's users and integrations.

## 5. How Custom Error Data Is Encoded

A custom error instance is encoded similarly to a function call with the same name and parameter types. Its revert data contains:

1. The first four bytes of the Keccak-256 hash of the error's canonical signature.
2. The ABI-encoded values of the error's arguments.

For example:

    error InsufficientBalance(uint256 available, uint256 required);

The canonical signature is:

    InsufficientBalance(uint256,uint256)

The selector is the first four bytes of the Keccak-256 hash of that signature. The complete revert payload is conceptually:

    selector (4 bytes)
    ++
    ABI-encoded available (32-byte word)
    ++
    ABI-encoded required (32-byte word)

For this particular error, the selector documented in Solidity's ABI example is <code>0xcf479181</code>. When decoding real data, compute or retrieve the selector from the exact canonical error signature instead of hard-coding selectors from memory.

The selector is only four bytes, so collisions are possible. Two different signatures can theoretically share the same four-byte selector; a decoder should use the expected ABI and context rather than assuming the selector uniquely identifies a globally unique error definition.

### Errors with no parameters

For a parameterless error such as <code>error Unauthorized();</code>, the custom error payload consists only of its four-byte selector. NatSpec comments can explain the meaning to developers and tools, but that documentation is not embedded in the four-byte payload itself.

### Built-in error formats

Solidity also defines built-in error encodings used by common failure forms:

- <code>Error(string)</code> — commonly used by revert strings and the string-message overload of <code>require</code>; selector <code>0x08c379a0</code>.
- <code>Panic(uint256)</code> — used for certain compiler-detected or language-defined panic conditions, such as arithmetic overflow under checked arithmetic; selector <code>0x4e487b71</code>.

Do not confuse these built-in formats with your contract's custom error declarations.

## 6. Custom Errors With require

The original custom-error pattern is widely compatible with Solidity 0.8.4 and later:

    if (amount > available) {
        revert InsufficientBalance(available, amount);
    }

Solidity 0.8.26 introduced support for a custom error as the second argument to <code>require</code>; this support reached the legacy code-generation pipeline in Solidity 0.8.27.

    require(
        amount <= available,
        InsufficientBalance(available, amount)
    );

This syntax is convenient, but there are compiler-version details worth knowing. A Solidity compiler bug disclosed in September 2026 affected a narrow case: code compiled through the IR-based pipeline (<code>viaIR: true</code>) passed a custom error to <code>require</code> using named arguments in an order different from the error declaration. The affected case required at least two arguments to be reordered and at least one of those reordered arguments not to be a string literal. Solidity 0.8.37 fixes the bug. Positional arguments and a standalone <code>if (...) revert MyError(...)</code> do not use the affected code path. On older compilers, keep named arguments in declaration order or use positional arguments, and pin the compiler version in the project.

There is another subtlety: the arguments to <code>require(condition, CustomError(...))</code> are evaluated regardless of whether the condition succeeds. Do not put expensive computations or functions with side effects in the custom-error argument expressions. A standalone <code>if (!condition) revert CustomError(...)</code> makes the failure-only construction explicit.

## 7. Decoding and Catching Custom Errors

A client that knows the contract ABI can inspect the revert data and decode the error selector and arguments. This is useful for wallet messages, tests, simulation tools, debuggers, and protocol integrations.

For example, if a call returns an error selector matching <code>InsufficientBalance(uint256,uint256)</code>, the client can decode the available and required amounts according to that ABI.

In Solidity, an external call failure can be handled with <code>try/catch</code>. Custom errors are not written as named typed <code>catch</code> clauses; use the raw-byte catch form and inspect the payload when custom-error-specific handling is necessary:

    try vault.withdraw(amount) {
        // Call succeeded.
    } catch (bytes memory lowLevelData) {
        // Inspect the selector and decode with the expected ABI.
    }

A <code>try/catch</code> applies to external calls made by the try expression. It does not let a function catch a revert from its own internal execution path as though it were an external call.

### Do not build critical control flow around arbitrary revert bytes

Revert data from a nested external call normally bubbles up through the caller chain. A contract may therefore receive an error defined by another contract it called, and a malicious callee can return bytes that imitate the selector and arguments of an error associated with a trusted interface.

Use decoded errors for diagnostics and appropriately designed handling, but do not treat revert data alone as proof that a particular contract or trusted component produced it. Validate the target, caller, state, and expected execution context separately.

## 8. Custom Errors and State Reversion

Custom errors explain why a path reverts, but they do not alter the fundamental rollback semantics of the EVM.

If a function makes state changes and then reverts with a custom error, state changes made in that reverted execution frame are rolled back. The error payload is returned to the caller. If a caller catches the failure of an external subcall, the caller can continue execution, but the state changes from the failed subcall remain reverted.

For example, a token transfer that reverts with <code>InsufficientBalance</code> must not leave behind the transfer's balance updates. The revert prevents those changes from being committed in the failed execution path.

Custom error data itself is not a stored contract record or an event log. It is failure return data associated with an execution. If a top-level transaction fails, the transaction receipt records failure status, while the full revert data is commonly available from transaction simulation, tracing, or RPC error responses—not as a normal event log emitted by the reverted transaction.

## 9. Standard Error Conventions: ERC-6093

A protocol may define its own error vocabulary, or it may choose standard errors when integrating with an existing token interface. **ERC-6093** specifies common custom errors for ERC-20, ERC-721, and ERC-1155 token implementations.

One example is:

    error ERC20InsufficientBalance(
        address sender,
        uint256 balance,
        uint256 needed
    );

Using standardized names and argument semantics can help wallets, developer libraries, and tests decode common token failures consistently. Standardization does not mean every deployed token implements these errors; applications must tolerate contracts that still use string reverts or other historical conventions.

For reusable libraries and protocol interfaces, choose error names and argument ordering deliberately. Changing an error's parameter types or order changes its canonical signature and selector, which can break tooling that relies on the old ABI.

## 10. Common Mistakes and Security Considerations

1. **Assuming custom errors are free.** They can reduce bytecode and revert-data costs, but reverting execution still consumes gas and error arguments must be encoded.
2. **Using vague error names.** Prefer a specific failure description such as <code>InsufficientBalance</code> over a generic <code>Failed</code>.
3. **Changing an error signature casually.** The canonical name and parameter types determine the selector; interface changes may break decoders and integrations.
4. **Treating a selector as globally unique.** Four-byte selectors can collide, so decode with the expected ABI and context.
5. **Trusting external revert data.** A nested or malicious callee can return bytes that imitate a custom error from another contract.
6. **Using error data as authentication.** An error selector does not prove who caused the failure or that a target is trusted.
7. **Using side-effecting expressions in custom-error arguments.** Error arguments may be evaluated before the failing branch is known, especially with the <code>require</code> custom-error overload.
8. **Ignoring compiler version behavior.** The custom-error form of <code>require</code> has version-specific support, and a compiler bug affecting named arguments was fixed in Solidity 0.8.37.
9. **Expecting a custom error to appear as an event.** Revert payloads and event logs are different mechanisms; failed execution does not preserve logs emitted only in reverted frames.
10. **Overloading or overriding errors.** Solidity custom errors cannot be overloaded or overridden. They are inherited, and identical signatures in different scopes produce indistinguishable selectors.

## 11. Key Takeaways

- Custom errors are named, typed failure descriptions declared with the <code>error</code> keyword.
- Use <code>revert MyError(args)</code> to stop execution and return structured error data.
- Custom error data consists of a four-byte selector followed by ABI-encoded arguments.
- Custom errors often reduce deployment bytecode and revert-data costs compared with long strings, but actual savings depend on usage.
- Solidity 0.8.26 added <code>require(condition, CustomError(...))</code>; legacy pipeline support arrived in 0.8.27.
- Solidity 0.8.37 fixed a reported IR code-generation bug affecting named-argument use in that <code>require</code> pattern; standalone <code>if (...) revert CustomError(...)</code> remains an explicit alternative.
- Error data can bubble up from nested calls and can be forged, so it is not proof of the source or a substitute for security checks.
- ERC-6093 standardizes commonly used token errors, but deployed token implementations may use different revert conventions.

## References

- [Solidity Documentation — Contracts: Custom Errors](https://docs.solidity.org/en/latest/contracts.html#custom-errors)
- [Solidity Documentation — ABI Specification: Errors](https://docs.solidity.org/en/latest/abi-spec.html#errors)
- [Solidity 0.8.4 — Custom Errors announcement](https://www.soliditylang.org/blog/2021/04/21/custom-errors/)
- [Solidity 0.8.26 Release Announcement — Custom errors in require](https://www.soliditylang.org/blog/2024/05/21/solidity-0.8.26-release-announcement/)
- [Solidity 0.8.27 Release Announcement — Legacy pipeline support](https://www.soliditylang.org/blog/2024/09/04/solidity-0.8.27-release-announcement/)
- [Solidity Security Alert — Named parameters in require with custom errors](https://www.soliditylang.org/blog/2026/09/10/misordered-named-parameters-in-require-with-custom-errors-bug/)
- [ERC-6093 — Custom Errors for Commonly-Used Tokens](https://eips.ethereum.org/EIPS/eip-6093)
- [Return Data and Revert Data](../01-evm-execution/10-return-data-and-revert-data.md)
- [Error Selectors](15-error-selectors.md)

## Review Checklist

- [ ] Explain how custom errors are declared and how reverting rolls back state changes.
- [ ] Describe the error selector and ABI-encoded argument layout.
- [ ] Compare custom errors with revert strings and built-in error formats.
- [ ] Explain compiler-version differences for custom errors in `require`, including the 0.8.37 fix.
- [ ] Explain external error bubbling, decoding, and why revert bytes can be spoofed.
- [ ] Consult the official Solidity documentation, compiler notes, and ERC-6093 where relevant.
