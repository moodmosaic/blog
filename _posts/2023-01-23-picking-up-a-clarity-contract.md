---
layout: basic
title: Picking up a Clarity contract
---

*This article is part of the Pandora [series of articles][series-of-articles].*

*Pandora enables property-based testing, fuzzing, and invariant testing for smart contracts that run on the Stacks 2.x layer-1 blockchain. Pandora discovers and run tests written in Clarity and TypeScript.*

[series-of-articles]: /bitcoin/stacks/pandora

## Cargo

Cargo is a fictional, simple smart contract that runs on the Stacks blockchain. Its purpose is to help facilitate tracking the progress a shipment makes throughout the supply chain.

As a smart contract, Cargo has the following properties:

* a user can create a new shipment
* a shipper can update the shipment status
* a user can not update another shipper's shipment status
* a user can read the current status of a shipment

You can find out more about Cargo in [this][kenny-rogers-article] article by [Kenny Rogers][kenny-rogers-twitter]. Cargo is deployed in Stacks testnet as [ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo][cargo].

[cargo]: https://explorer.stacks.co/txid/0x5864dabc9122732e16fcebd5ddaa727db8614eaee59499967c18011c1ddbd5b8?chain=testnet
[kenny-rogers-article]: https://dev.to/stacks/test-driven-stacks-development-with-clarinet-2e4i
[kenny-rogers-twitter]: https://twitter.com/kentherogers

What's *very* interesting--and it is the reason I am picking up this particular smart contract--is that the version of the Cargo smart contract deployed on testnet has a bug. (And yes, Cargo was [unit-tested][cargo-tests].)

[cargo-tests]: https://github.com/kenrogers/cargo/commit/63ae16ee84b03ff087f439498e489742fbf5fe68#diff-2978df20fa696c9a65fce8380d76aa9f2322db34fe4437821833fadab649cdd3

## Issue

`BUG` Can not read the status of past shipments, other than the last created one.<br>
`FIX` [Add the ID of the newly created entity to the internal map][cargo-bug-fix] that keeps track of the shipments.

[cargo-tests]: https://github.com/kenrogers/cargo/commit/63ae16ee84b03ff087f439498e489742fbf5fe68#diff-2978df20fa696c9a65fce8380d76aa9f2322db34fe4437821833fadab649cdd3
[cargo-bug-fix]: https://github.com/kenrogers/cargo/commit/758dbf51c5e43521032549b19d427467b7d2c195#diff-ddee0aadb9729d02051e6a8fd76e0f59e45cee0f37ba767ba2b91b4aeea46ff1

Next: Invariant testing from command-line.
