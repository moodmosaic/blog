---
layout: basic
title: Clarity "Hello, World!" deployed on Stacks
summary:
id-image: 3
tags:
    - Clarity
    - Stacks
---

[Clarity](https://clarity-lang.org/) is a relatively new language that brings smart contracts to Bitcoin.

- It is *decidable* which means programs must necessarily terminate
- It is interpreted and broadcasted on the network *as-is*

Clarity is being actively developed by [Stacks](https://www.stacks.co/), [Hiro](https://hiro.so/), & [Algorand](https://www.algorand.com/). — Here's how to deploy a "Hello, World!" smart contract on Stacks mainnet.

## Prerequisites

- Clarinet ^0.23.1
- Git ^2.33.0
- VS Code ^1.63.2 or Sublime Text ^4126 with [Clarity syntax highlighting](/2021/12/23/clarity-syntax-definitions-for-sublime-text/)

## Steps

<b>1. Create a new Clarinet project:</b>

```
$ clarinet new clarity-hello-world

Creating directory clarity-hello-world
Created file clarity-hello-world/contracts
Created file clarity-hello-world/settings
Created file clarity-hello-world/tests
Created file clarity-hello-world/Clarinet.toml
Created file clarity-hello-world/settings/Mainnet.toml
Created file clarity-hello-world/settings/Testnet.toml
Created file clarity-hello-world/settings/Devnet.toml
Created file clarity-hello-world/.vscode
Created file clarity-hello-world/.vscode/settings.json
Created file clarity-hello-world/.vscode/tasks.json
Created file clarity-hello-world/.gitignore
```

<b>2. Create a Git repository:</b>

```
$ cd clarity-hello-world
$ git init

Initialized empty Git repository in /clarity-hello-world/.git/
```

<b>3. Add the project on Git:</b>

```
$ git add .
$ git commit -m "Add empty project"

[master (root-commit) 1c57404] Add empty project
 5 files changed, 163 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 .vscode/settings.json
 create mode 100644 .vscode/tasks.json
 create mode 100644 Clarinet.toml
 create mode 100644 settings/Devnet.toml
```

<b>4. Add a new Clarity contract to the project:</b>

```
$ clarinet contract new hello-world

Created file contracts/hello-world.clar
Created file tests/hello-world_test.ts
Updated Clarinet.toml with contract hello-world
```

Clarinet added for us two files which you will discuss in a moment:

```
Created file contracts/hello-world.clar
Created file tests/hello-world_test.ts
```

Clarinet also modified the project file called `Clarinet.toml` in order to reflect the fact that you have added a new contract called hello-world:

```
Updated Clarinet.toml with contract hello-world
```

At this point let's pause and look at what Git shows us:

```
$ git status

On branch master
Changes not staged for commit:
 modified: Clarinet.toml
Untracked files:
 contracts/
 tests/
no changes added to commit (use "git add" and/or "git commit -a")
```

Now if you `git diff` inside `Clarinet.toml` you can see the actual changes:

```diff
$ git diff Clarinet.toml

diff --git a/Clarinet.toml b/Clarinet.toml
index 62aab1c..712cec9 100644
--- a/Clarinet.toml
+++ b/Clarinet.toml
@@ -1,10 +1,11 @@
 [project]
 name = "clarity-hello-world"
 authors = []
+description = ""
 telemetry = true
+requirements = []
 analysis = ["check_checker"]
+costs_version = 2
+[contracts.hello-world]
+path = "contracts/hello-world.clar"
+depends_on = []
```

Let's inform Git about our latest changes before moving on with the actual contract code:

```
$ git commit -m "Add contract hello-world"

[master 40a9650] Add contract hello-world
 3 files changed, 47 insertions(+), 4 deletions(-)
 create mode 100644 contracts/hello-world.clar
 create mode 100644 tests/hello-world_test.ts
```

To recap, those 3 files changed (mentioned in the Git output above) are the ones Clarinet touched:

```
Created file contracts/hello-world.clar
Created file tests/hello-world_test.ts
Updated Clarinet.toml with contract hello-world
```

<b>5. Add functions to the contract:</b>

Open `contracts/hello-world.clar`. It should look similar to this:

```
;; hello-world
;; <add a description here>
;; constants
;;
;; data maps and vars
;;
;; private functions
;;
;; public functions
;;
```

For this tutorial, you will add two Clarity functions to the contract.

## Add a public function

The first function is a public function called `say-hi`.

```
;; public functions
;;
(define-public (say-hi) (ok "hello world"))
```

What public means in Clarity is that the function is callable from other smart contracts.

The function doesn't take any parameters and simply returns "hello world" using the [`ok` response constructor](/2022/01/04/clarity-reference-part-1-primitives/#functions--impure).

## Add a read-only function

The second function is a [read-only](https://docs.stacks.co/references/language-functions#define-read-only) function called `echo-number`.

```
;; private functions
;;
(define-read-only (echo-number (val int)) (ok val))
```

What read-only means is that the function can't change internal state (variables, maps, etc).

This time the function takes an input parameter of type `int` and returns the value passed in using the `ok` response constructor.

The full `contracts/hello-world.clar` file should look like this:

```
;; hello-world
;; <add a description here>
;; constants
;;
;; data maps and vars
;;
;; private functions
;;
(define-read-only (echo-number (val int)) (ok val))
;; public functions
;;
(define-public (say-hi) (ok "hello world"))
```

Or in terms of `git diff`:

```diff
$ git diff contracts/hello-world.clar

diff --git a/contracts/hello-world.clar b/contracts/hello-world.clar
index 4f55c4d..faa51ef 100644
--- a/contracts/hello-world.clar
+++ b/contracts/hello-world.clar
@@ -10,6 +10,8 @@
;; private functions
 ;;
+(define-read-only (echo-number (val int)) (ok val))
;; public functions
 ;;
+(define-public (say-hi) (ok "hello world"))
```

Now you will run `clarinet check` in order to see whether the code you have added type-checks:

```
$ clarinet check

✔ Syntax of 1 contract(s) successfully checked
```

<b>6. Interact with the contract:</b>

In the project root directory `(clarity-hello-world)` launch the local Clarinet console.

When the Clarinet console is started, it shows a summary of the available contracts (in this case `hello-world`) and the simulated wallets in memory:

```
$ clarinet console

clarity-repl v0.21.0
Enter "::help" for usage hints.
Connected to a transient in-memory database.

Contracts
+---------------------------------------+-------------------------+
| Contract identifier                   | Public functions        |
+---------------------------------------+-------------------------+
| ST1PQHQKV0RJXZFY1DGX8M....hello-world | (echo-number (val int)) |
|                                       | (say-hi)                |
+---------------------------------------+-------------------------+
Initialized balances
+-----------------------------------------------+-----------------+
| Address                                       | STX             |
+-----------------------------------------------+-----------------+
| ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VG... (deployer) | 100000000000000 |
+-----------------------------------------------+-----------------+
| ST1SJ3DTE5DN7X54YDH5D64R3BCB6A2... (wallet_1) | 100000000000000 |
+-----------------------------------------------+-----------------+
| ST2CY5V39NHDPWSXMW9QDT3HC3GD6Q6... (wallet_2) | 100000000000000 |
+-----------------------------------------------+-----------------+
[...]
+-----------------------------------------------+-----------------+
| STNHKEPYEPJ8ET55ZZ0M5A34J0R3N5F... (wallet_9) | 100000000000000 |
+-----------------------------------------------+-----------------+
>>
```

This console is a Clarinet read-eval-print loop (REPL) that executes Clarity code instantly when a function is called.

The console provides the ability to interact with a contract using Clarity commands.

Call the `say-hi` function with the following command:

```
>> (contract-call? .hello-world say-hi)

(ok "hello world")
```

Next, call the `echo-number` function:

```
>> (contract-call? .hello-world echo-number 1337)

(ok 1337)
```

Next, try calling the echo-number function with an incorrect type, in this case an unsigned integer:

```
>> (contract-call? .hello-world echo-number u1337)

:1:42: error: expecting expression of type 'int', found 'uint'(contract-call? .hello-world echo-number u1337)
                                                                                                       ^~~~
```

Exit the Clarinet console/REPL by hitting CTRL-C twice. Then track on Git the latest changes:

```
$ git add contracts/hello-world.clar
$ git commit -m "Add public functions"

[master 1a3ee48] Add public functions
 1 file changed, 2 insertions(+)
```

<b>7. Deploy the contract on Stacks mainnet:</b>

Remember that in step 1 we got a bunch of files created by Clarinet

```
[...]

Created file clarity-hello-world/contracts
Created file clarity-hello-world/settings
Created file clarity-hello-world/tests
Created file clarity-hello-world/Clarinet.toml
Created file clarity-hello-world/settings/Mainnet.toml
Created file clarity-hello-world/settings/Testnet.toml
Created file clarity-hello-world/settings/Devnet.toml
Created file clarity-hello-world/.vscode
Created file clarity-hello-world/.vscode/settings.json
Created file clarity-hello-world/.vscode/tasks.json
Created file clarity-hello-world/.gitignore
```

Those files under settings control (among other things) the deployment process of the contracts.

For mainnet, edit `settings/Mainnet.toml` and provide the seed phrase of the deployer account:

```
[network]
name = "mainnet"
node_rpc_address = "https://stacks-node-api.mainnet.stacks.co"
deployment_fee_rate = 10

[accounts.deployer]
mnemonic = "<YOUR PRIVATE MAINNET MNEMONIC HERE>"
```

If there are no funds on the deployer account, Clarinet will throw an error:

```
$ clarinet publish --mainnet

{
   "error":"transaction rejected",
   "reason":"NotEnoughFunds",
   "reason_data":{
      "actual":"0x00000000000000000000000000000000",
      "expected":"0x00000000000000000000000000000910"
   },
   "txid":"85f71b70e724e5c79b116fc45623a5575c833709c28cfb4125301..."
}
```

Notice the `expected` value; that is the amount of network fee that must be paid:

```
HEX: 0x00000000000000000000000000000910
DEC:     2320
STX: 0.002320 (2320 / 100000)
```

Once there are funds in the deployer account, Clarinet can proceed with the publish:

```
$ clarinet publish --mainnet

Contract hello-world broadcasted in mempool
(txid: 85f71b70e724e5c79b116fc45623a5575c833709c28cfb4..., nonce: 0)
```

Here's the actual transaction as shown on the explorer: [0x85f71b70e724e5c79b116fc45623a5575c833709c28cfb4125301e9cf5f51a15](https://explorer.stacks.co/txid/0x85f71b70e724e5c79b116fc45623a5575c833709c28cfb4125301e9cf5f51a15?chain=mainnet)

## Summary

With Clarinet, deploying on Stacks mainnet is pretty straightforward:

1. Create a new Clarinet project
2. Create a Git repository for the project
3. Add the project on the Git repository
4. Add a new Clarity contract to the project
5. Add functions to the Clatity contract
6. Interact with the Clatity contract locally
7. Deploy the Clarity contract on Stacks mainnet

Next: [Clarity property-based testing primer](/2022/03/05/clarity-property-based-testing-primer/).
