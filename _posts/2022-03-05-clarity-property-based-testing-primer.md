---
layout: basic
title: Clarity property-based testing primer
---

Clarinet's `new` project command, as shown in [Clarity "Hello, World!"](https://blog.nikosbaxevanis.com/2022/01/28/clarity-hello-world-deployed-on-stacks/) post, creates the following files:

```
$ clarinet new clarity-hello-world

Creating directory clarity-hello-world
Created file clarity-hello-world/contracts
Created file clarity-hello-world/settings
Created file clarity-hello-world/tests
                                 ^^^^^
Created file clarity-hello-world/Clarinet.toml
Created file clarity-hello-world/settings/Mainnet.toml
Created file clarity-hello-world/settings/Testnet.toml
Created file clarity-hello-world/settings/Devnet.toml
Created file clarity-hello-world/.vscode
Created file clarity-hello-world/.vscode/settings.json
Created file clarity-hello-world/.vscode/tasks.json
Created file clarity-hello-world/.gitignore
```

Clarinet's `new` project command also creates a `test` folder, as shown with `^^^^^` above, which is where tests can exist once you add a Clarity contract to the project:

```
$ clarinet contract new hello-world

Created file contracts/hello-world.clar
Created file tests/hello-world_test.ts
                   ^^^^^^^^^^^^^^^^^^^
```

Once you add a Clarity contract to the project (*hello-world*, in this case), you can look inside `hello-world_test.ts` where you will see the anatomy of a Clarinet test:

```typescript
$ cat tests/hello-world_test.ts

import { Clarinet, Tx, Chain, Account, types }
  from 'https://deno.land/x/clarinet@v0.14.0/index.ts';

import { assert, assertEquals }
  from 'https://deno.land/std@0.90.0/testing/asserts.ts';

Clarinet.test({
  name: "Ensure that <...>",
  async fn(chain: Chain, accounts: Map<string, Account>) {
    // Arrange
    // Act
    // Assert
    assertEquals(true, true);
  }
});
```

>*I have formated, modified and simplified the body of the test above.*

If you run `clarinet test` you can see whether the test passes or not:

```diff
$ clarinet test

Running file:///.../clarity-hello-world/tests/hello-world_test.ts
* Ensure that <...> ... ok (21ms)
```

Change `assertEquals(true, true)` to `assertEquals(true, false)` above and rerun:

```diff
$ clarinet test

Running file:///.../clarity-hello-world/tests/hello-world_test.ts
* Ensure that <...> ... FAILED (16ms)

failures:

Ensure that <...>
AssertionError: Values are not equal:


    [Diff] Actual / Expected


-   true
+   false

    at assertEquals (https://deno.land/std@0.90.0/testing/asserts.ts:219:9)
    at Object.fn (file:///.../clarity-hello-world/tests/hello-world_test.ts:13:5)
    at fn (https://deno.land/x/clarinet@v0.14.0/index.ts:237:23)
    at asyncOpSanitizer (deno:runtime/js/40_testing.js:21:15)
    at resourceSanitizer (deno:runtime/js/40_testing.js:58:13)
    at exitSanitizer (deno:runtime/js/40_testing.js:85:15)
    at runTest (deno:runtime/js/40_testing.js:199:13)
    at Object.runTests (deno:runtime/js/40_testing.js:244:13)
    at file:///.../clarity-hello-world/$deno$test.js:1:27

failures:

        Ensure that <...>
```

>*We have seen the test passing, and now we have also seen the test failing.*

## Internals

Recall that once you add a Clarity contract to the project (*hello-world*, in this case), you can look inside `hello-world_test.ts` where you will see the anatomy of a Clarinet test:

```typescript
Clarinet.test({
  name: "Ensure that <...>",
  async fn(chain: Chain, accounts: Map<string, Account>) {
    // Arrange
    // Act
    // Assert
    assertEquals(true, true);
  }
});
```

>*Where do `chain` and `accounts` come from? How is `Clarinet.test` defined?*

Here's the (simplified, for this post) definition of [Clarinet.test](https://github.com/hirosystems/clarinet/blob/9d649801ac8610933b9ce1d4e9343323c18ef658/deno/index.ts#L214):

```typescript
type TestFunction = (
  chain    : Chain,
  accounts : Map<string, Account>,
  contracts: Map<string, Contract>
) => void | Promise<void>;

type PreSetupFunction = () => Array<Tx>;

interface UnitTestOptions {
  name     : string;
  preSetup?: PreSetupFunction;
  fn       : TestFunction;
}

export class Clarinet {
  static test(options: UnitTestOptions) {
    Deno.test({
      name: options.name,
      async fn() {
        var transactions: Array<Tx> = [];
        if (options.preSetup) {
          // These go into the Chain object.
          transactions = options.preSetup();
        }

        let chain     : Chain                 = ...
        let accounts  : Map<string, Account>  = ...
        let contracts : Map<string, Contract> = ...

        await options.fn(chain, accounts, contracts);
        //                 ↓       ↓          ↓
        //      async fn(chain, accounts, contracts) {
        //        // Arrange
        //        // Act
        //        // Assert
        //      }
      },
    });
  }
}
```

And here are some notes to take:

1. Clarinet.test calls into [Deno.test](https://github.com/denoland/deno/blob/bdc8006a362b4f95107a25ca816dcdedb7f44e4a/cli/dts/lib.deno.ns.d.ts#L342)
2. [UnitTestOptions](https://github.com/hirosystems/clarinet/blob/9d649801ac8610933b9ce1d4e9343323c18ef658/deno/index.ts#L187) are mapped into Deno.test [TestDefinition](https://github.com/denoland/deno/blob/bdc8006a362b4f95107a25ca816dcdedb7f44e4a/cli/dts/lib.deno.ns.d.ts#L153) args
3. You can pass an optional [PreSetupFunction](https://github.com/hirosystems/clarinet/blob/9d649801ac8610933b9ce1d4e9343323c18ef658/deno/index.ts#L185) with an array of transactions
4. [TestFunction](https://github.com/hirosystems/clarinet/blob/9d649801ac8610933b9ce1d4e9343323c18ef658/deno/index.ts#L180) acts as the test's data context

Armed with the above knowledge, we've just discovered that we can also pass `contracts` to the test, if and when needed:

```typescript
import { Clarinet, Tx, Chain, Account, Contract, types }
                                       ^^^^^^^^
  from 'https://deno.land/x/clarinet@v0.14.0/index.ts';

import { assert, assertEquals }
  from 'https://deno.land/std@0.90.0/testing/asserts.ts';

Clarinet.test({
  name: "Ensure that <...>",
  async fn(chain: Chain, accounts: Map<string, Account>, contracts: Map<string, Contract>) {
                                                         ^^^^^^^^^
    // Arrange
    // Act
    // Assert
    assertEquals(true, true);
  }
});
```

## The system under test

The [system under test](http://xunitpatterns.com/SUT.html) in this post is the [sup.clar](https://github.com/kenrogers/sup/blob/f77d63beb1d4849cf93f7270984f16b78b95c14b/backend/contracts/sup.clar), by [Kenny Rogers](https://twitter.com/kentherogers), that can be found in the [ Introduction to Full-Stack Web3 Development with Stacks](https://dev.to/krgrs/built-on-bitcoin-an-introduction-to-full-stack-web3-development-with-stacks-me9).

The system allows you to write a utf8-encoded message in exchange for some STX fee:

```
;; sup
;; Write a message in exchange for some STX fee.

;; constants
(define-constant receiver-address
  'STH87THSH5ENMSQ8MTJDSBVZWVPYFP46E0FP10EA)

;; data maps and vars
(define-data-var total-sups uint u0)
(define-map messages principal (string-utf8 500))

;; public functions
(define-read-only (get-sups)
  (var-get total-sups)
)

(define-read-only (get-message (who principal))
  (map-get? messages who)
)

(define-public (write-sup (message (string-utf8 500)) (price uint))
  (begin
    (try!
      (stx-transfer? price tx-sender receiver-address)
    )
    ;; #[allow(unchecked_data)]
    (map-set messages tx-sender message)
    (var-set total-sups (+ (var-get total-sups) u1))
    (ok "Sup written successfully")
  )
)
```

>*We have a counter called `total-sups`, a map called `messages` (key: [principal](/2022/01/04/clarity-reference-part-1-primitives/), value: utf8-encoded string), a getter called `get-sups`, and `write-sup` for adding new messages.*

## REPL

Let's interact with the contract through the REPL. This can help us encode use-cases to test-cases afterwards.

```
$ clarinet console

clarity-repl v0.23.0
Enter "::help" for usage hints.
Connected to a transient in-memory database.

Contracts
+--------------------------------------+---------------------------------+
| Contract identifier                  | Public functions                |
+--------------------------------------+---------------------------------+
| ST1PQHQKV0RJXZFY1DGX8MNSNYVE3....sup | (get-message (who principal))   |
|                                      | (get-sups)                      |
|                                      | (write-sup                      |
|                                      |     (message (string-utf8 500)) |
|                                      |     (price uint))               |
+--------------------------------------+---------------------------------+

Initialized balances
+------------------------------------------------------+-----------------+
| Address                                              | STX             |
+------------------------------------------------------+-----------------+
| ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM (deployer) | 100000000000000 |
+------------------------------------------------------+-----------------+
| ST1SJ3DTE5DN7X54YDH5D64R3BCB6A2AG2ZQ8YPD5 (wallet_1) | 100000000000000 |
+------------------------------------------------------+-----------------+
| ...                                                                    |
+------------------------------------------------------+-----------------+
| ST2JHG361ZXG51QTKY2NQCVBPPRRE2KZB1HR05NNC (wallet_9) | 100000000000000 |
+------------------------------------------------------+-----------------+

```

→ Calling `get-sups` returns `0`
```
>> (contract-call? .sup get-sups)
u0
```

→ Calling `get-message` returns `none`
```
>> (contract-call? .sup get-message 'ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM)
none
```

→ Calling `write-sup` with a message and a price returns `ok`
```
>> (contract-call? .sup write-sup u"Lorem ipsum dolor sit amet" u1)
(ok "Sup written successfully")

Events emitted
{
   "type":"stx_transfer_event",
   "stx_transfer_event":{
      "sender"   :"ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM",
      "recipient":"STH87THSH5ENMSQ8MTJDSBVZWVPYFP46E0FP10EA",
      "amount"   :"1"
   }
}
```

→ Calling `get-message` returns `some` (for the `sender` above)
```
>> (contract-call? .sup get-message 'ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM)
(some u"Lorem ipsum dolor sit amet")
```

→ Calling `get-sups` returns `1`
```
>> (contract-call? .sup get-sups)
u1
```

## Test cases

First make sure the basic imports are in place:

```typescript
import { Clarinet, Tx, Chain, Account, types }
  from 'https://deno.land/x/clarinet@v0.14.0/index.ts';

import { assert, assertEquals }
  from 'https://deno.land/std@0.90.0/testing/asserts.ts';
```

Now, based on the REPL session we just had, here are some test cases to write:

1. get-message returns none when write-sup is not called
2. write-sup returns expected string
3. write-sup increases total count by 1

<span>Test case 1: get-message returns none when write-sup is not called</span>

```typescript
Clarinet.test({
  name: 'get-message returns none when write-sup is not called',
  async fn(chain: Chain, accounts: Map<string, Account>) {
    let results = [...accounts.values()].map(account => {
      const who = types.principal(account.address);
      const msg = chain.callReadOnlyFn(
        'sup', 'get-message', [who], account.address);
      return msg.result;
    });

    assert(results.length > 0);
    results.forEach(msg => msg.expectNone());
  }
});
```

An example in REPL looks like this:

```
>> (contract-call? .sup get-message 'ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM)
none
```

Notice the *[...accounts.values()].map(account =>* part. It's because Clarinet gives you 1 or more accounts to test your contract, and we want to go through all of them:

```
+------------------------------------------------------+-----------------+
| Address                                              | STX             |
+------------------------------------------------------+-----------------+
| ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM (deployer) | 100000000000000 |
+------------------------------------------------------+-----------------+
| ST1SJ3DTE5DN7X54YDH5D64R3BCB6A2AG2ZQ8YPD5 (wallet_1) | 100000000000000 |
+------------------------------------------------------+-----------------+
| ...                                                                    |
+------------------------------------------------------+-----------------+
| ST2JHG361ZXG51QTKY2NQCVBPPRRE2KZB1HR05NNC (wallet_9) | 100000000000000 |
+------------------------------------------------------+-----------------+
```

<span>Test case 2: write-sup returns expected string</span>

```typescript
import fc
  from 'https://cdn.skypack.dev/fast-check';

Clarinet.test({
  name: 'write-sup returns expected string',
  async fn(chain: Chain, accounts: Map<string, Account>) {
    // Property-based test, runs 100 times by default.
    fc.assert(fc.property(
      // Generate pseudo-random 'lorem ipsum' string and a number.
      fc.lorem(), fc.integer(1, 100), (lorem: string, integer: number) => {
        // Arrange
        const deployer = accounts.get('deployer')!;
        const msg = types.utf8(lorem);
        const stx = types.uint(integer);

        // Act
        const block = chain.mineBlock([
          Tx.contractCall(
            'sup', 'write-sup', [msg, stx], deployer.address)
        ]);
        const result = block.receipts[0].result;

        // Assert
        result
          .expectOk()
          .expectAscii('Sup written successfully');
      })
    );
  }
});
```

An example in REPL looks like this:

```
>> (contract-call? .sup write-sup u"Lorem ipsum dolor sit amet" u1)
(ok "Sup written successfully")
```

Notice the *fc.assert(fc.property(* part. That's [fast-check](https://github.com/dubzzz/fast-check) which, by default, runs the test 100 times and each time it generates
* a pseudo-random *Lorem ipsum* string for the message to add
* a pseudo-random number betweem 1 and 100 for the STX fee to pay

<span>Test case 3: write-sup increases total count by 1</span>

```typescript
Clarinet.test({
  name: 'write-sup increases total count by 1',
  async fn(chain: Chain, accounts: Map<string, Account>) {
    // Property-based test, runs 100 times by default.
    fc.assert(fc.property(
      // Generate pseudo-random 'lorem ipsum' string and a number.
      fc.lorem(), fc.integer(1, 100), (lorem: string, integer: number) => {
        // Arrange
        const deployer = accounts.get('deployer')!;
        let startCount = chain.callReadOnlyFn(
          'sup', 'get-sups', [], deployer.address).result;

        const msg = types.utf8(lorem);
        const stx = types.uint(integer);

        // Act
        chain.mineBlock([
          Tx.contractCall(
            'sup', 'write-sup', [msg, stx], deployer.address)
        ]);

        // Assert
        const endCount = chain.callReadOnlyFn(
          'sup', 'get-sups', [], deployer.address).result;

        startCount = startCount.replace('u', ''); // u100 -> 100
        endCount.expectUint(Number(startCount) + 1);
      })
    );
  }
});
```

An example in REPL looks like this:

```
>> (contract-call? .sup get-sups)
u1

>> (contract-call? .sup write-sup u"Lorem ipsum dolor sit amet" u10)
(ok "Sup written successfully")

>> (contract-call? .sup get-sups)
u2
```

## Summary

* Clarity contracts can be property-based tested in a reasonable way.
* A robust test-suite next to a Clarity contract acts as a [feedback mechanism](https://blog.ploeh.dk/2011/04/29/Feedbackmechanismsandtradeoffs/) in addition to the newly added [check-checker](https://www.hiro.so/blog/new-safety-checks-in-clarinet).
* Clarinet tests are essentially Deno tests.
* Any library that works in Deno can also work in Clarinet, e.g. fast-check.

Get the [source code](https://gist.github.com/moodmosaic/ec138d99346e11a3416b5ee216b20033) on github.

Next: [Clarity model-based testing primer](/2022/03/15/clarity-clarity-model-based-testing-primer/).
