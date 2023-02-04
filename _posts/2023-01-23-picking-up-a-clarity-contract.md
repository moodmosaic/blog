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

Cargo was [tested][cargo-tests], but the following bug made its way into testnet[^1]:

[cargo-tests]: https://github.com/kenrogers/cargo/commit/63ae16ee84b03ff087f439498e489742fbf5fe68#diff-2978df20fa696c9a65fce8380d76aa9f2322db34fe4437821833fadab649cdd3

`BUG` Can not read the status of past shipments, other than the last created one.<br>
`FIX` [Add the ID of the newly created entity to the internal map][cargo-bug-fix] that keeps track of the shipments.

## Detecting the unexpected

Testing can decrease the number of defects but not remove all defects[^2]. The existing tests verify that a given function works as expected. What you want is one step further:

>Verify that *combinations* of functions in the smart contract work as expected.

This is a known technique, used by the Ethereum community, refered to as invariant testing[^3] and can be--and should be--also used when building in Stacks.

Next: [Invariant testing from command-line](/2023/01/30/invariant-testing-from-command-line).

<div>---</div>

[^1]: <small>Reported by LNow and resolved by Kenny Rogers in [this][cargo-bug-fix] commit.</small>
[^2]: <small>Program testing can be used to show the presence of bugs, but never to show their absence.--Dijkstra (1970) "[Notes On Structured Programming][dijkstra-notes]" (EWD249), Section 3 ("On The Reliability of Mechanisms"), corollary at the end.</small>
[^3]: <small>Model-based testing, also known as [invariant testing][dapp-readme] on the Ethereum [dapp.tools][dapp] and [forge][forge] communities, has its origins in [Haskell][haskell] ([QuickCheck][quickcheck]) and later in [Erlang][erlang] ([Quviq QuickCheck][erlang-quickcheck]). The technique is described in this [paper](https://research.chalmers.se/en/publication/232550).</small>

[cargo-bug-fix]: https://github.com/kenrogers/cargo/commit/758dbf51c5e43521032549b19d427467b7d2c195#diff-ddee0aadb9729d02051e6a8fd76e0f59e45cee0f37ba767ba2b91b4aeea46ff1
[dijkstra-wiki]: https://en.wikipedia.org/wiki/Edsger_W._Dijkstra
[dijkstra-notes]: http://www.cs.utexas.edu/users/EWD/ewd02xx/EWD249.PDF
[dapp-readme]: https://github.com/dapphub/dapptools/blob/b4876106a5f4b263f8cf20b24a514a70e2326c86/src/dapp/README.md#invariant-testing
[dapp]: https://dapp.tools
[forge]: https://github.com/foundry-rs/foundry
[haskell]: https://www.haskell.org/
[quickcheck]: https://hackage.haskell.org/package/QuickCheck
[erlang]: https://www.erlang.org
[erlang-quickcheck]: http://www.quviq.com/products/erlang-quickcheck
