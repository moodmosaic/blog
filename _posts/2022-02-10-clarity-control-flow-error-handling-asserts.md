---
layout: basic
title: Clarity Control flow & error handling - asserts!
---

[Clarity of Mind](https://book.clarity-lang.org/) book discusses [control flow and error handling](https://book.clarity-lang.org/ch06-00-control-flow.html). Here's an example of how it looks like to refactor from `if` statements to [`asserts!`](https://book.clarity-lang.org/ch06-01-asserts.html).

In the book, [Marvin Janssen](https://github.com/MarvinJanssen) starts with a function called `is-valid-caller`, discussed in the [chapter on private functions](https://book.clarity-lang.org/ch05-02-private-functions.html).

In that chapter, there's an `if` function to only allow the action (add/delete a recipient) if the `tx-sender` is equal to the principal that deployed the contract:

```
(define-public (add-recipient (recipient principal) (amount uint))
    (if (is-valid-caller)
        (ok (map-set recipients recipient amount))
        err-invalid-caller
    )
)
(define-public (delete-recipient (recipient principal))
    (if (is-valid-caller)
        (ok (map-delete recipients recipient))
        err-invalid-caller
    )
)
```

Then, in the [`asserts!`](https://book.clarity-lang.org/ch06-01-asserts.html) chapter the code has been refactored to use `asserts!` instead of `if`.

Putting those two pieces in a git repository, you can see a *diff* highlighting the refactored part, from `if` to `asserts!`:

```diff
@@ -11,20 +11,22 @@
 (define-private (is-valid-caller)
     (is-eq contract-owner tx-sender)
 )

 (define-public (add-recipient (recipient principal) (amount uint))
-    (if (is-valid-caller)
+    (begin
+        ;; Assert the tx-sender is valid.
+        (asserts! (is-valid-caller) err-invalid-caller)
         (ok (map-set recipients recipient amount))
-        err-invalid-caller
     )
 )

 (define-public (delete-recipient (recipient principal))
-    (if (is-valid-caller)
+    (begin
+        ;; Assert the tx-sender is valid.
+        (asserts! (is-valid-caller) err-invalid-caller)
         (ok (map-delete recipients recipient))
-        err-invalid-caller
     )
 )

 ;; Two example calls to the public functions:
 (print (add-recipient 'ST1J4G6RR643BCG8G8SR6M2D9Z9KXT2NJDRK3FBTK u500))
```

Hope this helps the readers of the book to visualize the actual change.
