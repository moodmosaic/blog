---
layout: basic
title: Invariant testing from command-line
---

*This article is part of the Pandora [series of articles][series-of-articles].*

*Pandora enables property-based testing, fuzzing, and invariant testing for smart contracts that run on the Stacks 2.x layer-1 blockchain. Pandora discovers and run tests written in Clarity and TypeScript.*

[series-of-articles]: /bitcoin/stacks/pandora

In the [first article in this series][part1] you learned about Cargo, a fictional, simple smart contract that runs on the Stacks blockchain. Cargo had a bug that the existing tests could not detect.

[part1]: /2023/01/23/picking-up-a-clarity-contract

In this article you will learn how to uncover the bug using a technique called invariant testing, using only the command-line. Later in this series you will also use TypeScript and Clarity.

## Scaffolding

Get [Clarinet](https://www.hiro.so/clarinet), if not already installed, and use it to scaffold a new project named cargo:

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

Clarinet [won't create a Git repository yet](https://github.com/hirosystems/clarinet/issues/366) and so you have to do that manually:

```bash
$ cd cargo && git init && git add . && git status && git commit -m "Initial commit"

Initialized empty Git repository in /snapshot/dev/Pandora,_and_friends/cargo/.git/
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

Cargo is deployed as [ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo][cargo], if you recall the [first article in this series][part1]. You will interact with the Cargo contract using Clarinet's deployment plans:

[cargo]: https://explorer.stacks.co/txid/0x5864dabc9122732e16fcebd5ddaa727db8614eaee59499967c18011c1ddbd5b8?chain=testnet

```bash
$ clarinet requirements add ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo

Updated Clarinet.toml with requirement ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo
```

It's always a good practise to run `git diff` and see the changes that are made:

```diff
diff --git a/Clarinet.toml b/Clarinet.toml
index f295b7e..80083de 100644
--- a/Clarinet.toml
+++ b/Clarinet.toml
@@ -8,6 +8,8 @@ cache_dir = "./.cache"

 # [contracts.counter]
 # path = "contracts/counter.clar"
+[[project.requirements]]
+contract_id = 'ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo'

 [repl.analysis]
 passes = ["check_checker"]
```

Add any changes in the working directory to the staging area and commit:

```bash
$ git add Clarinet.toml && git commit -m "ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo"

[master f051fb4] ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo
 1 file changed, 2 insertions(+) # If you also see N deletions(-) it's a known issue:
                                 # https://github.com/hirosystems/clarinet/issues/417
```

## REPL

In the same, root, folder execute `clarinet console` to open the REPL. This will load the Cargo contract and generate 10 accounts with enough STX balance to make calls:

```bash
$ clarinet console

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
# 7 more addresses with STX balance         |                 |
+-------------------------------------------+-----------------+

>>
```

The next step is to interact with the Cargo contract through the REPL. This can help encode use-cases to test-cases in the next article in this series.

## Smart contract invariants

A smart contract invariant is a logical statement about the contract's observable, instantaneous state, that can be ensured by its public functions.

In REPL, randomly call public functions on the Cargo contract to build a simplified model. You will then compare that model's state with the actual public, observable, state of the Cargo contract:

![image](https://nikosbaxevanis.com/images/articles/2023-01-30-invariant-testing-from-command-line-1.png)
<small style="display: block;text-align: center;">Image taken and modified from [Spotify's article](https://engineering.atspotify.com/2015/06/rapid-check/) on the same topic.</small>

The simplified model in this case can be a dictionary of IDs (Key) and Shipments (Value). In its empty state it looks like this:

```
+-----+------------------------------------------------------------------------------------+
| ID  | Shipment                                                                           |
+-----+------------------------------------------------------------------------------------+
|     |                                                                                    |
+-----+------------------------------------------------------------------------------------+
```

### Unknown IDs

`u0`, `u1`, `u123` are all unknown IDs since they don't exist in the model and in the Cargo contract's state yet. The expected behavior is to have the Cargo contract return an error:

```bash
>> (contract-call? 'ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo get-shipment u1)

(status "Does not exist"))
```

Current state of the simplified model stays the same:

```
+-----+------------------------------------------------------------------------------------+
| ID  | Shipment                                                                           |
+-----+------------------------------------------------------------------------------------+
|     |                                                                                    |
+-----+------------------------------------------------------------------------------------+
```

(The state of the simplified model stays the same also for ID `u0` and ID `u123` and any other integer.)

### Create a shipment

With location `"Athens"` and receiver address `ST1SJ3DTE5DN7X54YDH5D64R3BCB6A2AG2ZQ8YPD5` the Cargo contract returns a response indicating success:

```bash
>> (contract-call? 'ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo create-new-shipment
  "Athens" 'ST1SJ3DTE5DN7X54YDH5D64R3BCB6A2AG2ZQ8YPD5)

(ok "Shipment created successfully")
```

Current state of the simplified model is updated:

```
+-----+------------------------------------------------------------------------------------+
| ID  | Shipment                                                                           |
+-----+------------------------------------------------------------------------------------+
|  1  | location "Athens"                                                                  |
|     | receiver ST1SJ3DTE5DN7X54YDH5D64R3BCB6A2AG2ZQ8YPD5                                 |
+-----+------------------------------------------------------------------------------------+
```

### Sanity check 1

With ID `u1`, `get-shipment` should return a response matching with the entry for ID `1` in current state of the simplified model:

```bash
>> (contract-call? 'ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo get-shipment u1)

(location "Athens") (receiver ST1SJ3DTE5DN7X54YDH5D64R3BCB6A2AG2ZQ8YPD5) ...
```

### Create another shipment

With location `"Prague"` and receiver address `ST2CY5V39NHDPWSXMW9QDT3HC3GD6Q6XX4CFRK9AG` the Cargo contract returns a response indicating success:

```bash
>> (contract-call? 'ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo create-new-shipment
  "Prague" 'ST2CY5V39NHDPWSXMW9QDT3HC3GD6Q6XX4CFRK9AG)

(ok "Shipment created successfully")
```

Current state of the simplified model is updated:

```
+-----+------------------------------------------------------------------------------------+
| ID  | Shipment                                                                           |
+-----+------------------------------------------------------------------------------------+
|  1  | location "Athens"                                                                  |
|     | receiver ST1SJ3DTE5DN7X54YDH5D64R3BCB6A2AG2ZQ8YPD5                                 |
+-----+------------------------------------------------------------------------------------+
|  2  | location "Prague"                                                                  |
|     | receiver ST2CY5V39NHDPWSXMW9QDT3HC3GD6Q6XX4CFRK9AG                                 |
+-----+------------------------------------------------------------------------------------+
```

### Sanity check 2

With ID `u2`, `get-shipment` should return a response matching with the entry for ID `2` in current state of the simplified model:

```bash
>> (contract-call? 'ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo get-shipment u2)

(status "Does not exist"))
```

```bash
Expected: (location "Prague") (receiver ST2CY5V39NHDPWSXMW9QDT3HC3GD6Q6XX4CFRK9AG) ...
  Actual: (status "Does not exist"))
```

### Sanity check 3

With ID `u1`, `get-shipment` should return a response matching with the entry for ID `1` in current state of the simplified model:

```bash
>> (contract-call? 'ST3QFME3CANQFQNR86TYVKQYCFT7QX4PRXM1V9W6H.cargo get-shipment u1)

(location "Prague") (receiver ST2CY5V39NHDPWSXMW9QDT3HC3GD6Q6XX4CFRK9AG) ...
```

```bash
Expected: (location "Athens") (receiver ST1SJ3DTE5DN7X54YDH5D64R3BCB6A2AG2ZQ8YPD5) ...
  Actual: (location "Prague") (receiver ST2CY5V39NHDPWSXMW9QDT3HC3GD6Q6XX4CFRK9AG) ...
```

## Sanity check automation

With sanity checks `2` and `3` you were able to uncover the bug in the Cargo contract:

`BUG` Can not find past shipments.<br>
`FIX` [Store the ID of the newly added internally][cargo-bug-fix].

[cargo-bug-fix]: https://github.com/kenrogers/cargo/commit/758dbf51c5e43521032549b19d427467b7d2c195#diff-ddee0aadb9729d02051e6a8fd76e0f59e45cee0f37ba767ba2b91b4aeea46ff1

Next: Invariant testing from TypeScript.
