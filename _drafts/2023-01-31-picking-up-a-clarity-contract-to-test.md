---
layout: basic
title: Picking up a Clarity contract to test
---

*This article is part 1 of the Pandora [series of articles][series-of-articles] about deploying on Stacks with confidence.*

[series-of-articles]: /bitcoin/stacks/pandora

## Cargo

Cargo is a fictional, simple smart contract that runs on the Stacks blockchain. Its purpose is to help facilitate tracking the progress a shipment makes throughout the supply chain.

Cargo smart contract has the following properties:

* a user can create a new shipment
* a shipper can update the shipment status
* a user can not update another shipper's shipment status
* a user can read the current status of a shipment

You can find out more about Cargo in [this][kenny-rogers-article] article by [Kenny Rogers][kenny-rogers-twitter]. Cargo is deployed in Stacks testnet as [ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo][cargo].

[cargo]: https://explorer.stacks.co/txid/0x5864dabc9122732e16fcebd5ddaa727db8614eaee59499967c18011c1ddbd5b8?chain=testnet
[kenny-rogers-article]: https://dev.to/stacks/test-driven-stacks-development-with-clarinet-2e4i
[kenny-rogers-twitter]: https://twitter.com/kentherogers

What's *very* interesting--and it is the reason I am picking up this particular smart contract--is that the version of the Cargo smart contract deployed on testnet has a bug.

But, was Cargo unit-tested? Yes, Cargo has a [test-suite][cargo-tests] with a test for each property.

[cargo-tests]: https://github.com/kenrogers/cargo/commit/63ae16ee84b03ff087f439498e489742fbf5fe68#diff-2978df20fa696c9a65fce8380d76aa9f2322db34fe4437821833fadab649cdd3

## Cargo bug

```bash
$ clarinet new cargo

Created directory cargo
Created directory cargo/contracts
Created directory cargo/settings
Created directory cargo/tests
Created file cargo/Clarinet.toml
Created file cargo/settings/Mainnet.toml
Created file cargo/settings/Testnet.toml
Created file cargo/settings/Devnet.toml
Created directory cargo/.vscode
Created file cargo/.vscode/settings.json
Created file cargo/.vscode/tasks.json
Created file cargo/.gitignore
```

```bash
$ cd cargo && git init && git add . && git status && git commit -m "Initial commit"

Initialized empty Git repository in C:/Snapshot/dev/Pandora,_and_friends/cargo/.git/
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   .gitignore
        new file:   .vscode/settings.json
        new file:   .vscode/tasks.json
        new file:   Clarinet.toml
        new file:   settings/Devnet.toml

[master (root-commit) 4940253] Initial commit
 5 files changed, 195 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 .vscode/settings.json
 create mode 100644 .vscode/tasks.json
 create mode 100644 Clarinet.toml
 create mode 100644 settings/Devnet.toml

```

```bash
$ clarinet requirements add ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo

Updated Clarinet.toml with requirement ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo
```

```bash
$ git add Clarinet.toml && git commit -m "ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo"

[master d4e531d] ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo
 1 file changed, 12 insertions(+), 15 deletions(-)
```

```bash
$ clarinet console

clarity-repl v1.4.0
Enter "::help" for usage hints.
Connected to a transient in-memory database.
+-------------------------------------------------+-------------------------------------------+
| Contract identifier                             | Public functions                          |
+-------------------------------------------------+-------------------------------------------+
| ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo | (create-new-shipment                      |
|                                                 |     (starting-location (string-ascii 25)) |
|                                                 |     (receiver principal))                 |
|                                                 | (get-shipment (shipment-id uint))         |
|                                                 | (update-shipment                          |
|                                                 |     (shipment-id uint)                    |
|                                                 |     (current-location (string-ascii 25))) |
+-------------------------------------------------+-------------------------------------------+

+-------------------------------------------+-----------------+
| Address                                   | uSTX            |
+-------------------------------------------+-----------------+
| ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM | 100000000000000 |
+-------------------------------------------+-----------------+
| ST1SJ3DTE5DN7X54YDH5D64R3BCB6A2AG2ZQ8YPD5 | 100000000000000 |
+-------------------------------------------+-----------------+
| ST2CY5V39NHDPWSXMW9QDT3HC3GD6Q6XX4CFRK9AG | 100000000000000 |
+-------------------------------------------+-----------------+
| ST2JHG361ZXG51QTKY2NQCVBPPRRE2KZB1HR05NNC | 100000000000000 |
+-------------------------------------------+-----------------+
| ST2NEB84ASENDXKYGJPQW86YXQCEFEX2ZQPG87ND  | 100000000000000 |
+-------------------------------------------+-----------------+
| ST2REHHS5J3CERCRBEPMGH7921Q6PYKAADT7JP2VB | 100000000000000 |
+-------------------------------------------+-----------------+
| ST3AM1A56AK2C1XAFJ4115ZSV26EB49BVQ10MGCS0 | 100000000000000 |
+-------------------------------------------+-----------------+
| ST3NBRSFKX28FQ2ZJ1MAKX58HKHSDGNV5N7R21XCP | 100000000000000 |
+-------------------------------------------+-----------------+
| ST3PF13W7Z0RRM42A8VZRVFQ75SV1K26RXEP8YGKJ | 100000000000000 |
+-------------------------------------------+-----------------+
| STNHKEPYEPJ8ET55ZZ0M5A34J0R3N5FM2CMMMAZ6  | 100000000000000 |
+-------------------------------------------+-----------------+

>>
```

```bash
>> (contract-call? 'ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo get-shipment u1)

(tuple (status "Does not exist"))
```

```bash
>> (contract-call? 'ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo create-new-shipment "Athens" 'STNHKEPYEPJ8ET55ZZ0M5A34J0R3N5FM2CMMMAZ6)

(ok "Shipment created successfully")
```

```bash
>> (contract-call? 'ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo get-shipment u1)

(tuple (location "Athens") (receiver STNHKEPYEPJ8ET55ZZ0M5A34J0R3N5FM2CMMMAZ6) (shipper ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM) (status "In Transit"))
```

```bash
>> (contract-call? 'ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo create-new-shipment "Prague" 'STNHKEPYEPJ8ET55ZZ0M5A34J0R3N5FM2CMMMAZ6)

(ok "Shipment created successfully")
```

```bash
>> (contract-call? 'ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo get-shipment u2)

(tuple (status "Does not exist"))
```

```bash
>> (contract-call? 'ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo get-shipment u1)

(tuple (location "Prague") (receiver STNHKEPYEPJ8ET55ZZ0M5A34J0R3N5FM2CMMMAZ6) (shipper ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM) (status "In Transit"))
```
