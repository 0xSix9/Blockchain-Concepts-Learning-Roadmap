# Mappings and Dynamic Arrays in Storage: Hash-Derived Locations

> **Stage:** 03 — Storage and Data Layout  
> **Priority:** 1  
> **Status:** Reviewed  
> **Last reviewed:** 2026-10-17

## 1. Overview

Solidity mappings and dynamically sized arrays have a different storage layout from ordinary state variables. Instead of placing every entry directly after the declaration, Solidity reserves a **base slot** and uses Keccak-256-based calculations to derive where the entries or elements live.

The main rules are:

- A **mapping** reserves a base slot for layout purposes, but the slot itself does not hold the mapping's key-value entries.
- A **dynamic array** normally stores its current length in its base slot, while its element data begins at a slot derived from the base slot.
- **Nested mappings and arrays** apply the location rules recursively.
- **bytes and string** are special: short values are stored inline in their own slot, while longer values use a separate data area.

This is important because a developer may know the declaration and key yet still need to calculate a slot to inspect or decode the raw storage. It is also critical for proxies and upgrades: if a mapping or array's base slot changes, the derived locations can change too.

This article focuses on deriving locations. [Storage Layout](01-storage-layout.md) explains how declarations receive base slots; [Storage Slots](02-storage-slots.md) covers raw slot reads; [Storage Packing](03-storage-packing.md) explains how small values share words.

![Diagram showing mapping values derived from a key and base slot, dynamic array elements derived from the array base slot, and bytes/string short versus long storage](../../assets/diagrams/mappings-dynamic-arrays-storage-overview.svg)

## 2. Why a Base Slot Is Needed

Assume a contract contains:

```solidity
mapping(address => uint256) balances; // base slot p
uint256[] values;                     // base slot q
```

The compiler assigns each declaration a storage position according to the contract's full layout, including inheritance. Call these base positions p and q.

The contents of a mapping cannot be laid out as a simple consecutive list because the set of keys is not known at compile time and may be very sparse. A dynamic array can grow, so reserving a fixed consecutive range at the declaration position would not work either.

Instead, Solidity derives locations with Keccak-256:

- For a mapping entry, the key and the mapping's base slot are combined and hashed.
- For a dynamic array, the hash of its base slot marks the start of its data area.

The hashing rules make the derived locations depend on the declaration's base slot. They also make separate mappings or arrays use different areas even when their keys or indexes happen to be the same.

**Do not guess p or q from a variable's position in a snippet.** In a real contract, preceding state variables, inheritance, custom storage bases, and layout changes can affect the base slot. Verify it from the compiler's storage-layout output.

## 3. Mapping Storage: Key Plus Base Slot

For a mapping located at base slot p, the value associated with key k is stored at a slot derived from:

```text
keccak256(h(k) || encode32(p))
```

Here 