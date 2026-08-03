---
title: Why some reverse functions are slow
---

I overheard two students discussing "the time complexity of reverse," without
any more concrete referent. This post is a response for those students.

If you ask whether `reverse` is fast, the right first response is: which
`reverse`?

There is a mathematical function called reverse. It takes a list like
`'(1 2 3)` and produces `'(3 2 1)`. But there are many algorithms that compute
that function. Some are linear time. Some are quadratic time. The result is the
same; the amount of work can be very different.

Here are two Racket programs that both reverse a list:

```racket
(define (rev1 l)
  (cond
    [(empty? l) '()]
    [(cons? l)
     (append (rev1 (rest l))
             (list (first l)))]))

(define (rev2 l)
  (rev2-help l '()))

(define (rev2-help l acc)
  (cond
    [(empty? l) acc]
    [(cons? l)
     (rev2-help (rest l)
                (cons (first l) acc))]))
```

They compute the same answers:

```racket
> (rev1 '(1 2 3 4))
'(4 3 2 1)

> (rev2 '(1 2 3 4))
'(4 3 2 1)
```

But `rev1` is slower for large lists. The reason is not mysterious once we look
at what each step is forced to do.

Throughout, the complexity measure is time as a function of the length of the
input list. When we say `O(n)` or `O(n^2)`, `n` is the number of elements in the
list being reversed.

## Lists Are One-Way

A Racket list is built out of pairs. Each pair points at the next part of the
list. That makes some operations cheap:

```racket
(first '(1 2 3)) ; 1
(rest  '(1 2 3)) ; '(2 3)
(cons 0 '(1 2 3)) ; '(0 1 2 3)
```

Each of those operations can be treated as constant time for this discussion.
They look at, or add, the front of the list.

But `append` is different. To append one list onto another, Racket has to walk
down the first list. For example:

```racket
(append '(1 2 3) '(4 5))
```

The result is `'(1 2 3 4 5)`, but getting there requires visiting the elements
of `'(1 2 3)`. If the first list has length `m`, the append takes time
proportional to `m`.

That one fact explains the difference between the two versions of `reverse`.

## The Direct Version

The direct version says:

```racket
(define (rev1 l)
  (cond
    [(empty? l) '()]
    [(cons? l)
     (append (rev1 (rest l))
             (list (first l)))]))
```

In English:

1. Reverse the rest of the list.
2. Put the first element at the end.

For a small list, that is perfectly clear:

```racket
(rev1 '(1 2 3 4))
= (append (rev1 '(2 3 4)) '(1))
= (append (append (rev1 '(3 4)) '(2)) '(1))
= (append (append (append (rev1 '(4)) '(3)) '(2)) '(1))
= '(4 3 2 1)
```

The expensive part is hidden in "put the first element at the end." Since lists
are easy to add to at the front, but not at the back, `rev1` uses `append`.

For a list of length 4, the append work has this shape:

```text
append a list of length 0
append a list of length 1
append a list of length 2
append a list of length 3
```

For a list of length `n`, the append work has this shape:

```text
0 + 1 + 2 + ... + (n - 1)
```

That sum is the familiar triangular number:

```text
0 + 1 + 2 + ... + (n - 1) = n(n - 1) / 2
```

The important part is the `n^2`. The exact constants do not matter for big-O
analysis, so this version is `O(n^2)`.

We can also say this with a recurrence. Let `R(n)` be the cost of reversing a
list of length `n` with `rev1`. Let `A(m)` be the cost of appending a first list
of length `m`. These costs are still measured in terms of list length: `R`
varies with the length of the list being reversed, and `A` varies with the
length of the first argument to `append`.

```text
R(0) = constant
R(n) = R(n - 1) + A(n - 1) + constant
```

Since `A(n - 1)` is linear in `n - 1`, `R(n)` accumulates a linear amount of
append work at each level of recursion. Adding those linear pieces together
gives the quadratic sum above.

## The Accumulator Version

The second version uses a helper function:

```racket
(define (rev2 l)
  (rev2-help l '()))

(define (rev2-help l acc)
  (cond
    [(empty? l) acc]
    [(cons? l)
     (rev2-help (rest l)
                (cons (first l) acc))]))
```

This program carries an accumulator, `acc`, that represents the part of the
answer built so far. At each step it removes one element from the front of the
input list and adds that element to the front of the accumulator.

Here is the state of the computation for `'(1 2 3 4)`:

| Remaining input | Accumulator |
| --- | --- |
| `'(1 2 3 4)` | `'()` |
| `'(2 3 4)` | `'(1)` |
| `'(3 4)` | `'(2 1)` |
| `'(4)` | `'(3 2 1)` |
| `'()` | `'(4 3 2 1)` |

When the input is empty, the accumulator is the reversed list.

The key operation is `cons`, not `append`. Adding one item to the front of a
list is constant time. That means each recursive step does a constant amount of
work, then moves to a list that is one element shorter. The accumulator grows,
but the cost of `cons` does not grow with it.

If `H(n)` is the cost of `rev2-help` on a list of length `n`, then:

```text
H(0) = constant
H(n) = H(n - 1) + constant
```

After `n` steps, that is just:

```text
H(n) = constant * n + constant
```

So `rev2-help` is `O(n)`, and `rev2` is also `O(n)`.
Again, this is linear in the length of the original input list.

## The Lesson

It is imprecise to say "the complexity of `reverse`" without saying which
algorithm we mean. The function being computed is not the whole story.

`rev1` and `rev2` produce the same lists, but `rev1` repeatedly pays for
`append`, and those costs add up:

```text
0 + 1 + 2 + ... + (n - 1)
```

That is why `rev1` is quadratic.

`rev2` does one constant-time `cons` per input element. That is why `rev2` is
linear in the length of the list being reversed.

The built-in `reverse` you expect from a language implementation should be the
linear-time kind. But if you write your own, the difference between "put it on
the end" and "carry an accumulator" is the difference between an algorithm that
slows down quadratically and one that scales directly with the size of the list.
