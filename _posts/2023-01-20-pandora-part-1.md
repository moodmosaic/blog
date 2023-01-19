---
layout: basic
title: Pandora - Part 1
---

*Pandora allows you to test your smart contracts in both Clarity and TypeScript. It also enables property-based testing and headless fuzzing. This is Part 1 of the [article series](/2023/01/16/pandora/).*

A typical workflow in the Stacks ecosystem is declaring functions in Clarity and then writing tests in TypeScript. As an example, you define in Clarity a `get-counter` read-only function

```
(define-data-var counter uint u0)

(define-read-only (get-counter)
  (var-get counter)
)
```

and then you write a test in TypeScript for your Clarity function (or vice-versa if you are doing Test-Driven Development)

```
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

Clarity can be a first-class citizen when testing smart contracts. Being able to write tests in Clarity helps you minimize context switching, so you can focus on what matters.

It should be possible to write tests in Clarity as [Jude Nelson](https://fosstodon.org/@judecnelson) very nicely does [here](https://github.com/jcnelson/poxl/blob/c4d66d035170e67c7d8a9c6a0c0662d378dcd077/tests/test-poxl.clar). Here's a snippet:

```
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

In a Clarity contract, any function that meets the following criteria is essentially a test cadidate:

* The function has a certain type of visibility (defualt *private*, configurable)
* The function name starts with *xyz* (default *test-*, configurable)
* If a function named *beforeEach* exists, it can be executed before each test is run

In both cases (tests written in TypeScript and tests written in Clarity) when a test function takes parameters, these can be auto-generated using a [fuzzer](https://blog.nelhage.com/post/property-testing-is-fuzzing) and then the test can run multiple times (default *250*, configurable).
