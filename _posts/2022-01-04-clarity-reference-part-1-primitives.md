---
layout: basic
title: Clarity reference - Part 1 / Primitives
summary:
id-image: 3
tags:
    - Clarity
    - Stacks
---

Clarity differs from most other smart contract languages in two important ways:

- It is *decidable* which means programs must necessarily terminate
- It is interpreted and broadcasted on the network *as-is*

## Contracts composition

- Data — the space which only the smart contract itself may modify
- Functions — operate within the data space of the smart contract

## Language features

- No recursion
- No lambdas
- No mutation
- Loops only via `fold`, `map` and `filter`
- Atomic semantics on booleans, integers, [principals](https://docs.stacks.co/write-smart-contracts/principals), [buffers](https://docs.stacks.co/write-smart-contracts/values#buffers)
- Functions, *pure* (read-only) and *impure* (modify state)

## Functions / pure

- Specified via `define-read-only`
- Return any type
- No state mutations — result in an error otherwise

## Functions / impure

- Specified via `define-public`
- Return a `Response` type result where
  -  `ok` means the function is considered valid
  - `err` means the function is considered invalid

Let's assume that function `A` calls function `B`

- `A` returns `err`, `B` returns `ok`
  - no effects from `A`
  - no effects from `B`
- `A` returns `ok`, `B` returns `err`
  - effects from `A`
  - no effects from `B`
- `A` returns `ok`, `B` returns `ok`
  - effects from `A`
  - effects from `B`

## Support for lists

- Multi-dimensional, of the same type
- `fold`, `map` and `filter` on lists
- Variable length when used as function inputs
  - An extra constant parameter required that indicates the maximum output size when used as function outputs

Next: [Clarity reference — Part 2 / Data-Space Primitives](/2022/01/17/clarity-reference-part-2-data-space-primitives/).
