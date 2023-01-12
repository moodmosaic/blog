---
layout: basic
title: Clarity reference - Part 2 / Data-Space Primitives
---

Data within a smart contract’s data-space is stored within `maps`.

`maps` associate a pair of two values `(a, b)` (also known as [tuple](https://en.wikipedia.org/wiki/Tuple)) to another tuple. It works similar to a [name-value pair](https://en.wikipedia.org/wiki/Name%E2%80%93value_pair).

To get or set a value in a map you use:

1. `(map-get map-name key-tuple)` — This fetches the value associated with a given key in the map, or returns `none` if there is no such value.
2. `(map-set! map-name key-tuple value-tuple)` — This will set the value of `key-tuple` in the data map
3. `(map-insert! map-name key-tuple value-tuple)` — This will set the value of `key-tuple` in the data map if and only if an entry does not already exist.
4. `(map-delete! map-name key-tuple)` — This will delete `key-tuple` from the data map.

A smart contract defines the data schema of a data map through `define-map`.

## Primitive Types

Clarity is a strongly-typed language. Its type system has the following types:

- `(tuple (key-name-0 key-type-0) (key-name-1 key-type-1) ...)` — a typed tuple with named fields.
- `(list max-len entry-type)` — a list of maximum length `max-len`, with entries of type `entry-type`.
- `(response ok-type err-type)` — used by public functions to commit their changes or abort.
- `(optional some-type)` — an option type for objects that can either be `(some value)` or `none`.
- `principal` — object representing a principal. A principal represents the address on the blockchain. There are two types of principals; standard principal (has private key) and contract principal (points to a smart contract).
- `bool` — boolean value (`true` or `false`)
- `int` — signed 128-bit integer
- `uint` — unsigned 128-bit integer
- “unknown types” — the type system does not generally allow them

Next: [Clarity “Hello, World!” deployed on Stacks](/2022/01/28/clarity-hello-world-deployed-on-stacks/).
