---
layout: basic
title: Pandora - Part 1
---

Pandora enables property-based testing and headless fuzzing for your Clarity contracts. It allows you to write your tests in both Clarity and TypeScript. Read the introduction post [here](/2023/01/16/pandora/).

## Write tests in TypeScript

A typical workflow in the Stacks ecosystem is declaring functions in Clarity and then writing tests in TypeScript. For example, you define in Clarity a `get-counter` [read-only function](https://book.clarity-lang.org/ch05-03-read-only-functions.html)

```
(define-data-var counter uint u0)

(define-read-only (get-counter)
  (var-get counter)
)
```

and then you write a test in TypeScript for your Clarity function, or vice-versa if you are doing Test-Driven Development

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

As [discussed](https://blog.nikosbaxevanis.com/2022/03/05/clarity-property-based-testing-primer), these are essentially Deno tests.

This has the interesting side-effect of being able to use what's already available in Deno ecosystem. For example, [fast-check](https://github.com/dubzzz/fast-check) and [dspec](https://deno.land/x/dspec@v0.2.0) both can be used, as shown [here](https://github.com/moodmosaic/testing-example/commit/c02aaad9c6e10e7a1a62758dc83f4aab3b8a3c36).

## Write tests in Clarity

Clarity should be a first-class citizen when testing Clarity code. The syntax is similar to LISP, and not all LISP developers know - or want to know - about Deno, TypeScript and JavaScript.

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

Detecting those test functions and running them is doable; any function that meets the following criteria is essentially a test cadidate:

* The function has a certain visibility (private, public) - *configurable*
* The function name starts with `test` - *configurable*
* If a function named `beforeEach` exists, it can be executed before each test is run
* If a test function takes parameters, these can be auto-generated using a [fuzzer](https://blog.nelhage.com/post/property-testing-is-fuzzing) and then the test can run multiple times (default 250) - *configurable*
