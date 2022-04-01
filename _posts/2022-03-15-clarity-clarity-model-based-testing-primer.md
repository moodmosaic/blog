---
layout: basic
title: Clarity model-based testing primer
summary:
id-image: 3
tags:
    - Clarity
    - Stacks
---

In the previous post you saw how [Clarity contracts can be property-based tested in a reasonable way](/2022/03/05/clarity-property-based-testing-primer/).

We wrote property-based tests for the contract (sup.clar) verifying that *any* given function works as expected.

>[sup.clar](https://github.com/kenrogers/sup/blob/f77d63beb1d4849cf93f7270984f16b78b95c14b/backend/contracts/sup.clar), by [Kenny Rogers](https://twitter.com/kentherogers), can be found in the [ Introduction to Full-Stack Web3 Development with Stacks](https://dev.to/krgrs/built-on-bitcoin-an-introduction-to-full-stack-web3-development-with-stacks-me9).

In this post we're going one step further by verifying that *any combination* of functions defined in the contract works as expected.

This technique is known as *model-based testing* and has its origins in Haskell (QuickCheck) and later in Erlang (Quviq QuickCheck). The technique is described in this [paper](https://research.chalmers.se/en/publication/232550).

## The system under test

The [system under test](http://xunitpatterns.com/SUT.html) in this post is the same as in the [previous post](/2022/03/05/clarity-property-based-testing-primer/#the-system-under-test). It allows you to write a utf8-encoded message in exchange for some STX fee. We call this *the **real** system*.

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

We will create now a *TypeScript **model*** and compare the execution of that model with that of the real system (contract). That is, TypeScript model vs the actual contract.

## TypeScript model

[Clarinet tests are essentially Deno tests](/2022/03/05/clarity-property-based-testing-primer/#internals) hence the model will be written in TypeScript.

First make sure the basic imports are in place:

```typescript
// @ts-nocheck
// https://github.com/dubzzz/fast-check/issues/2781

import { Clarinet, Tx, Chain, Account, types }
  from 'https://deno.land/x/clarinet@v0.27.0/index.ts';

import fc
  from 'https://cdn.skypack.dev/fast-check';
```

Then map the contract in TypeScript in the form of [commands](https://dubzzz.github.io/fast-check/interfaces/Command.html) as defined in [fast-check](https://github.com/dubzzz/fast-check).

### WriteSupsCommand

The `write-sup` function of the contract
  * increments *total-sups* by 1
  * associates the given message with the caller's Stack address
  * returns *ok "Sup written successfully"*

Thus we define a `WriteSupsCommand` on the model to
  * keep track of the increments of *total-sups* by 1
  * keep track of the association of the given message with the caller's Stack address
  * verify that *ok "Sup written successfully"* is returned

```typescript
class WriteSupsCommand implements fc.Command<Model, Real> {
  readonly who: Principal;
  readonly msg: Utf8;
  readonly fee: Uint;

  constructor(who: Principal, msg: Utf8, fee: Uint) {
    this.who = who;
    this.msg = msg;
    this.fee = fee;
  }

  check(_: Readonly<Model>): bool {
    // Can always write a message in exchange for a STX fee.
    return true;
  }

  run(m: Model, real: Real): void {
    const block = real.chain.mineBlock([
      Tx.contractCall(
        'sup', 'write-sup', [this.msg.toString(), this.fee.toString()], this.who.asString())
    ]);
    const actual = block.receipts[0].result;

    actual
      .expectOk()
      .expectAscii('Sup written successfully');
    m.messages[this.who] = this.msg;
    m.supCount = m.supCount + 1;

    console.log(
      `for Ӿ${this.fee.asString()} ✎ ${this.msg.asString()}, by ${this.who.asString()}`);
  }
}
```

### GetSupsCommand

The `get-sups` function of the contract
  * returns the *total-sups* count

Thus we define a `GetSupsCommand` on the model to
  * verify that the *expected* count (the one we keep track in `WriteSupsCommand`) equals the actual *total-sups* returned from the contract

```typescript
class GetSupsCommand implements fc.Command<Model, Real> {
  readonly who: Principal;

  constructor(who: Principal) {
    this.who = who;
  }

  check(_: Readonly<Model>): bool {
    // Can always check total-sups.
    return true;
  }

  run(m: Model, real: Real): void {
    const msg = real.chain.callReadOnlyFn(
      'sup', 'get-sups', [], this.who.asString());
    const actual = msg.result;

    actual.expectUint(m.supCount);
  }
}
```

### GetMessageCommand

The `get-message` function of the contract
  * takes a [Principal](https://docs.stacks.co/write-smart-contracts/principals) as argument and returns the associated *message* for it

Thus we define a `GetMessageCommand` on the model to
  * verify that the *expected* message (the one we keep track of in `WriteSupsCommand`) equals the actual message returned from the contract

```typescript
class GetMessageCommand implements fc.Command<Model, Real> {
  readonly who: Principal;

  constructor(who: Principal) {
    this.who = who;
  }

  check(m: Readonly<Model>): bool {
    // Can get message if there is one.
    return m.messages[this.who] !== undefined;
  }

  run(m: Model, real: Real): void {
    const msg = real.chain.callReadOnlyFn(
      'sup', 'get-message', [this.who.toString()], this.who.asString());
    const actual = msg.result;

    actual
      .expectSome()
      .expectUtf8(m.messages[this.who].asString());
  }
}
```

## Where is the state actually stored?

There are two custom types, one for the model and one the contract:

→ `Model` holds the state of the system under test.

```typescript
type Model = {
  messages: Map<Principal, Utf8>,
  supCount: number
};
```

→ `Real` holds the state of the actual system that's stored on the blockchain.

```typescript
type Real = {
  chain: Chain
};
```

## How TypeScript types map into Clarity types?

The commands reference some custom types. `Principal`, `Utf8`, and `Uint`, are all helpers for marshalling between Clarity and TypeScript.

## Principal
```typescript
class Principal {
  readonly value: string;

  constructor(value: string) {
    this.value = value;
  }

  toString(): string {
    return types.principal(this.value);
  }

  asString(): string {
    return this.value;
  }
}
```

Sample output from REPL session:
```
> const principal =
    new Principal("ST2NEB84ASENDXKYGJPQW86YXQCEFEX2ZQPG87ND");

> principal.toString();
"'ST2NEB84ASENDXKYGJPQW86YXQCEFEX2ZQPG87ND"

> principal.asString();
"ST2NEB84ASENDXKYGJPQW86YXQCEFEX2ZQPG87ND"
```

### Utf8
```typescript
class Utf8 {
  readonly value: string;

  constructor(value: string) {
    this.value = value;
  }

  toString(): string {
    return types.utf8(this.value);
  }

  asString(): string {
    return this.value;
  }
}
```

Sample output from REPL session:
```
> const utf8 =
    new Utf8("Sup written successfully");

> utf8.toString();
'u"Sup written successfully"'

> utf8.asString();
"Sup written successfully"
```

### Uint
```typescript
class Uint {
  readonly value: number;

  constructor(value: number) {
    this.value = value;
  }

  toString(): string {
    return types.uint(this.value);
  }

  asString(): string {
    return this.value.toString();
  }
}
```

Sample output from REPL session:
```
> const uint =
    new Uint(100);

> uint.toString();
"u100"

> uint.asString();
"100"
```

## Bringing it all together

```typescript
Clarinet.test({
  name: 'sup.clar stateful property-based testing',
  async fn(chain: Chain, accounts: Map<string, Account>) {
    const commands = [
      // Create GetSupsCommand.
      fc.constantFrom(...accounts.values())
        .map(account =>
          new GetSupsCommand(
            new Principal(account.address))),

      // Create GetMessageCommand.
      fc.constantFrom(...accounts.values())
        .map(account =>
          new GetMessageCommand(
            new Principal(account.address))),

      // Create WriteSupsCommand.
      fc.record({
          who: fc.constantFrom(...accounts.values()).map(account => account.address)
        , msg: fc.lorem()
        , fee: fc.integer(10, 99)
      }).map(r =>
          new WriteSupsCommand(
            new Principal(r.who), new Utf8(r.msg), new Uint(r.fee)))
    ];
    const model = {
      messages: new Map <Principal, Utf8>(),
      supCount: 0
    };

    fc.assert(fc.property(
      // Generate a random command sequence.
      fc.commands(commands, { size: '+1' }), (commands) => {
        const initialState = () => ({ model: model,  real: { chain: chain } });
        fc.modelRun(initialState, commands);
    }), { numRuns: 10 }); // Run `numRuns` times.
  }
});
```

If we [run `clarinet test`](/2022/03/15/clarity-clarity-model-based-testing-primer/) we can see whether the test passes or not. Here's a sample output:
```
Running file:///.../sup/tests/sup_test.ts
for Ӿ91 ✎ pulvinar, by ST3NBRSFKX28FQ2ZJ1MAKX58HKHSDGNV5N7R21XCP
for Ӿ58 ✎ amet a enim integer adipiscing, by ST2REHHS5J3CERCRBEPMGH7921Q6PYK...
for Ӿ97 ✎ mauris nunc, by ST1SJ3DTE5DN7X54YDH5D64R3BCB6A2AG2ZQ8YPD5
for Ӿ63 ✎ eros scelerisque vitae massa massa, by ST2JHG361ZXG51QTKY2NQCVBPPR...
for Ӿ71 ✎ convallis adipiscing sodales, by ST2CY5V39NHDPWSXMW9QDT3HC3GD6Q6XX...
for Ӿ76 ✎ in varius cras ut, by ST2JHG361ZXG51QTKY2NQCVBPPRRE2KZB1HR05NNC
for Ӿ37 ✎ elit, by ST2NEB84ASENDXKYGJPQW86YXQCEFEX2ZQPG87ND
for Ӿ60 ✎ congue nulla libero, by ST1SJ3DTE5DN7X54YDH5D64R3BCB6A2AG2ZQ8YPD5
for Ӿ18 ✎ rutrum magna et justo, by ST3AM1A56AK2C1XAFJ4115ZSV26EB49BVQ10MGCS0
for Ӿ65 ✎ integer massa, by ST2REHHS5J3CERCRBEPMGH7921Q6PYKAADT7JP2VB
for Ӿ56 ✎ mauris mollis id rhoncus a, by STNHKEPYEPJ8ET55ZZ0M5A34J0R3N5FM2CM...
for Ӿ73 ✎ velit, by ST2NEB84ASENDXKYGJPQW86YXQCEFEX2ZQPG87ND
for Ӿ86 ✎ fermentum mollis vitae mi, by ST2JHG361ZXG51QTKY2NQCVBPPRRE2KZB1HR...
for Ӿ28 ✎ enim pellentesque, by STNHKEPYEPJ8ET55ZZ0M5A34J0R3N5FM2CMMMAZ6
for Ӿ48 ✎ ultrices nec sed justo, by ST2REHHS5J3CERCRBEPMGH7921Q6PYKAADT7JP2VB
for Ӿ11 ✎ fermentum in mauris, by ST3PF13W7Z0RRM42A8VZRVFQ75SV1K26RXEP8YGKJ
for Ӿ84 ✎ sollicitudin adipiscing ut, by ST1SJ3DTE5DN7X54YDH5D64R3BCB6A2AG2Z...
for Ӿ60 ✎ purus, by ST3PF13W7Z0RRM42A8VZRVFQ75SV1K26RXEP8YGKJ
for Ӿ75 ✎ orci ipsum pellentesque, by ST2CY5V39NHDPWSXMW9QDT3HC3GD6Q6XX4CFRK9AG
for Ӿ21 ✎ tempus quis non congue dolor, by ST2NEB84ASENDXKYGJPQW86YXQCEFEX2Z...
for Ӿ18 ✎ orci pellentesque lorem purus nulla, by STNHKEPYEPJ8ET55ZZ0M5A34J0...
for Ӿ63 ✎ aliquet suspendisse nunc, by ST2JHG361ZXG51QTKY2NQCVBPPRRE2KZB1HR0...
for Ӿ79 ✎ velit egestas, by ST3AM1A56AK2C1XAFJ4115ZSV26EB49BVQ10MGCS0
for Ӿ10 ✎ eu, by ST3PF13W7Z0RRM42A8VZRVFQ75SV1K26RXEP8YGKJ
for Ӿ95 ✎ sed pharetra metus quis vel, by ST3AM1A56AK2C1XAFJ4115ZSV26EB49BVQ...
* sup.clar stateful property-based testing ... ok (1013ms)
```

Get the [source code](https://gist.github.com/moodmosaic/ec138d99346e11a3416b5ee216b20033#file-sup_test_model-ts) on github.

## Summary

* Model-based testing as in [Haskell](https://www.haskell.org/) [QuickCheck](https://github.com/nick8325/quickcheck), [Hedgehog](https://github.com/hedgehogqa/haskell-hedgehog), and [Erlang QuickCheck](http://www.quviq.com/products/erlang-quickcheck/) is possible on Stacks and Clarity, thanks to [fast-check](https://github.com/dubzzz/fast-check) and [Clarinet](https://github.com/hirosystems/clarinet) from [Hiro](https://www.hiro.so/).
* The idea of the approach is to define commands that could be applied to a Clarity contract. Pick zero, one or more commands and run them sequentially if they can be executed on the current state.
* Model-based testing requires a model. A model is a simplified version of the contract. We define a TypeScript model representing the contract.
* Commands are defined for each one of the public functions of the contract. They come with two methods:
  * `check(m: Readonly<Model>): boolean` to allow or not allow the command to be executed given the current state.
  * `run(m: Model, r: Real): void` to execute the command on the contract and update the model accordingly.
    * Also checks for potential problems or inconsistencies between the model and the contract.
* Finally [do not write (manual, end-to-end) tests, generate them!](https://www.youtube.com/watch?v=hXnS_Xjwk2Y) Bugs can be hard to catch, often they can rely on scenarios which are not the mainstream.

*If you are eager to see a similar example in Haskell Hedgehog, here's [a similar example of model-based testing between Haskell and C#](https://github.com/moodmosaic/hedgehog-inline-csharp-testing) I've worked on a few years back.*
