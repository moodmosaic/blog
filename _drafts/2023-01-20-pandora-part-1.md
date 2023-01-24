---
layout: basic
title: Pandora - Part 1
---

*Pandora enables property-based testing and headless-fuzzing for smart contracts that run on the Stacks 2.x layer-1 blockchain. This is Part 1 of the [article series](/2023/01/16/pandora/).*

A typical workflow in the Stacks ecosystem is declaring functions in Clarity and then writing tests in TypeScript. As an example, you define in Clarity a `get-counter` function

```lisp
(define-data-var counter uint u0)

(define-read-only (get-counter)
  (var-get counter)
)
```

and then you write a test in TypeScript for that function (or vice-versa if you are doing Test-Driven Development)

```ts
Clarinet.test({
  name: "ensure <get-counter> send the counter value",
  fn(chain: Chain, accounts: Map<string, Account>) {
    const { address } = accounts.get("wallet_1")!;
    const block = chain.mineBlock([
      Tx.contractCall("counter", "get-counter", [], address),
    ]);

    block.receipts[0].result.expectUint(0);
  },
});
```

These TypeScript tests [are essentially Deno tests](https://blog.nikosbaxevanis.com/2022/03/05/clarity-property-based-testing-primer). This means you can use what's already available in Deno ecosystem. For example, [fast-check](https://github.com/dubzzz/fast-check) and [dspec](https://deno.land/x/dspec@v0.2.0) both can be used, as we've done [here](https://github.com/moodmosaic/testing-example/commit/c02aaad9c6e10e7a1a62758dc83f4aab3b8a3c36) with _lnow_.

## Clarity

Clarity can be a first-class citizen when testing smart contracts. Being able to write tests in Clarity helps you minimize context switching, so you can focus on what matters.

It should be possible to write tests in Clarity as [Jude Nelson](https://fosstodon.org/@judecnelson) very nicely does [here](https://github.com/jcnelson/poxl/blob/c4d66d035170e67c7d8a9c6a0c0662d378dcd077/tests/test-poxl.clar). Here's a snippet:

```lisp
(define-private (test-get-block-commit-total)
    (begin
        (print "test-get-block-commit-total")
        (asserts! (is-eq u0 (get-block-commit-total (list ))) (err u0))
        (asserts! (is-eq u6 (get-block-commit-total
            (unwrap-panic (as-max-len? (list
                { miner: 'SP1GYBXAJSEF8SY0ERKA068J93E3EGNTXHR98MM5P, amount-ustx: u1 }
                { miner: 'SP2M85H4NNNPQB0Y7GHT3K5EHWMRZWTHF2QAY1W69, amount-ustx: u2 }
                { miner: 'SP3A33QYJK76BCDJJD11RYWZP9D62PVQXK2VF5TJN, amount-ustx: u3 }
            ) u32))))
            (err u0))
        (ok u0)
    )
)
```

In a Clarity contract, any function that meets the following criteria can essentially be a test cadidate:

* The function has a certain type of visibility (defualt *private*, configurable)
* The function name starts with *xyz* (default *test-*, configurable)
* If a function named *beforeEach* exists, it can be executed before each test is run

## Fuzzing

In both cases (tests written in TypeScript and tests written in Clarity) when a test function takes extra arguments, these can be auto-generated using a [fuzzer](https://blog.nelhage.com/post/property-testing-is-fuzzing). Then, the test can run multiple times (default *250*, configurable).

#### Step 1: Take an existing Clarinet test

```ts
Clarinet.test({
  name: 'write-sup returns expected string',
  async fn(chain: Chain, accounts: Map<string, Account>) {
    // Arrange
    const account = accounts.get('deployer')!;
    const msg = types.utf8("lorem ipsum");
    const stx = types.uint(123);

    // Act
    const block = chain.mineBlock([
      Tx.contractCall(
        'sup', 'write-sup', [msg, stx], account.address)
    ]);
    const result = block.receipts[0].result;

    // Assert
    result
      .expectOk()
      .expectAscii('Sup written successfully');
  }
});
```

#### Step 2: Change `test` to `fuzz` (or `prop`, to be discussed)

```diff
-Clarinet.test({
+Clarinet.fuzz({
   name: 'write-sup returns expected string',
   async fn(chain: Chain, accounts: Map<string, Account>) {
     // Arrange
     const account = accounts.get('deployer')!;
     const msg = types.utf8("lorem ipsum");
```

#### Step 3: Specify number of runs (optional - default is 250)

```diff
 Clarinet.fuzz({
   name: 'write-sup returns expected string',
+  runs: 10,
   async fn(chain: Chain, accounts: Map<string, Account>) {
     // Arrange
     const account = accounts.get('deployer')!;
     const msg = types.utf8("lorem ipsum");
     const stx = types.uint(123);
```

#### Step 4: Auto-generate values instead of hardcoding them

```diff
 Clarinet.fuzz({
   name: 'write-sup returns expected string',
   runs: 10,
-  async fn(chain: Chain, accounts: Map<string, Account>) {
+  async fn(chain: Chain, account: Account, message: string, howMuch: number|bigint) {
     // Arrange
-    const account = accounts.get('deployer')!;
-    const msg = types.utf8("lorem ipsum");
-    const stx = types.uint(123);
+    const msg = types.utf8(message);
+    const stx = types.uint(howMuch);

     // Act
     const block = chain.mineBlock([
       Tx.contractCall(
         'sup', 'write-sup', [msg, stx], account.address)
```

The test should now look like below:

```ts
Clarinet.fuzz({
  name: 'write-sup returns expected string',
  runs: 10,
  async fn(chain: Chain, account: Account, message: string, howMuch: number|bigint) {
    // Arrange
    const msg = types.utf8(message);
    const stx = types.uint(howMuch);

    // Act
    const block = chain.mineBlock([
      Tx.contractCall(
        'sup', 'write-sup', [msg, stx], account.address)
    ]);
    const result = block.receipts[0].result;

    // Assert
    result
      .expectOk()
      .expectAscii('Sup written successfully');
  }
});
```

Output:

```bash
$ clarinet test
=> returns expected val ... #1  wallet_9 12 ipsum semper ultricies ante p ... ok (17ms)
=> returns expected val ... #2  wallet_3 98 ultricies dolor pharetra a ... ok (13ms)
=> returns expected val ... #3  wallet_1 97 tempor erat ... ok (21ms)
=> returns expected val ... #4  wallet_3 99 ipsum consectetuer orci ... ok (15ms)
=> returns expected val ... #5  wallet_8 27 molestie eros ... ok (15ms)
=> returns expected val ... #6  wallet_5 13 risus enim aliquam aliquam ... ok (15ms)
=> returns expected val ... #7  wallet_8 16 vivamus ut ... ok (10ms)
=> returns expected val ... #8  wallet_2 46 suscipit consectetur non et ... ok (8ms)
=> returns expected val ... #9  wallet_9 55 dolor ... ok (10ms)
=> returns expected val ... #10 wallet_6 88 tincidunt ... ok (12ms)
=> returns expected val ... ok (164ms)

ok | 1 passed (10 steps) | 0 failed (218ms)
```
