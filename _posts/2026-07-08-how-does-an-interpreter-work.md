---
title: How does an interpreter work?
---

It sometimes helps to step through a non-trivial example of evaluation in order to
understand how an environment dynamically develops during the evaluation. We will
take this program as our running example:

```racket
((lambda (i)
   ((lambda (j)
      ((lambda (j)
         i)
       5))
    6))
 7)
```

We will step through the execution of this program, but first let us acquaint
ourselves with what the program actually does.

## The Outer Application

At the top level, this is an application. It is the application of the operator
expression

```racket
(lambda (i)
  ((lambda (j)
     ((lambda (j) i) 5))
   6))
```

to the operand `7`. In an application form, we call the first expression the
**[operator](https://www.youtube.com/watch?v=3RA4MykPm4s)** and the second
expression the operand. The operator is a lambda expression. If I were unsure of
that, we could write a program over our lambda-calculus datatype:

```racket
(define (primary-form expr)
  (match expr
    [`,y #:when (symbol? y) 'variable]
    [`(lambda (,x) ,body) 'lambda]
    [`(,rator ,rand) 'application]))
```

You can use this to verify the program for yourself. The body of that lambda
expression is also an application:

```racket
((lambda (j)
   ((lambda (j)
      i) ; this will be the point we discuss most
    5))
 6)
```

There are three useful things to notice. First, this is again an application of
a lambda whose body is also an application. Second, the ultimate value of this
whole big expression is going to come from the value of `i`. Third, `i` is a
free variable reference in

```racket
((lambda (j)
   ((lambda (j) i) 5))
 6)
```

This latter fact should not be surprising. If `x` occurs in the scope of an
expression, such as in `(lambda (x) (lambda (y) x))`, then that is a bound
reference to `x`, since the variable declaration of `x` in the lambda expression
is the referent of that bound variable reference.

If we restrict our focus to the body of that same expression, `(lambda (y) x)`,
then there must be a free variable reference to `x` in the body. That outer
`(lambda (x) ...)` is what will bind that free variable.

## Environments Remember Context

We perform evaluation of our programs via structural recursion, the same way we
write most other functions. That means the place where we actually want to get
the value of a variable is when we work our way down to the variable line. For
the program above, that happens when we have worked our way all the way down to
`i`.

By then, it is too late to figure out the value of `i` all by itself, because we
no longer have the information we need to determine what `i` means. Our solution
is to accumulate that information as we recur into the expression, and then look
up the meaning of bound variables in our accumulator when we reach a variable
case.

Since there are potentially scads and scads of bound variables in an expression,
we need to track each one differently. When we see `i`, we need to know to look
up `i` and not `x`. We also need to track not just the name of the variable, but
also its meaning. Our accumulator must therefore track two pieces for each
variable: the name, or formal parameter, of the lambda expression, and the value,
or actual argument, to which that formal parameter corresponds in this context.

When do we finally have both of those pieces? Look at the application line:

```racket
(match expr
  ...
  [`(,rator ,rand) ((valof rator env) (valof rand env))])
```

The `rator` has to evaluate to a Racket procedure, since we are about to call it
like a procedure of one argument. We are going to call that procedure on the
value of the `rand`. The `rator` and `rand` can both be big honkin'
expressions, but ultimately, when we get values for them and pass the results
back up the recursion, the value of the `rator` had better be a procedure. The
value of the `rand` can be any old value.

Only after we finish evaluating the two sub-pieces of the `rator`/`rand` form,
and then do the Racket application, do we have both the formal and actual
parameters at the same time.

## Lambda Bodies Wait

Here is a simpler example of something like our interpreter:

```racket
((lambda (b) (if b 5 6)) (not false))
```

We first evaluate the operator and operand to values: the former to a Racket
procedure and the latter to a Racket boolean. Then we call the Racket procedure
on the Racket boolean. Only after that can we evaluate the body `(if b 5 6)`.

That is an important point: we never evaluate the body of a lambda expression
until after we have applied it to a value.

```racket
(define loop
  (lambda (x)
    (loop x)))
```

The reason you can get by with writing down a definition like `loop` is that we
do not evaluate the body of the expression unless and until we invoke it with an
argument.

## Extending The Environment

It is on the lambda line that we actually extend our environment. The right-hand
side of the lambda line is:

```racket
(lambda (a)
  (valof body (lambda (y)
                (if (eqv? x y)
                    a
                    (env y)))))
```

That is the function we invoke with `true` or `7` or whatever value we have in
our example. It is not until we invoke that function that we begin to evaluate
the body. As soon as we recur in to evaluate the body, we will no longer have
access to the surrounding `(lambda (x) ...)`, because that was the point of
recurring down into the smaller expression.

To keep that information with us as we go, we add it to our accumulator, called
an environment. We are using a functional accumulator; we can just as well look
at our interpreter as a mathematical description of a particular function from
variables to values.

This step, adding to the accumulator, is also called **extending** the function.
If you think of a function as a set of pairs, then we have added one more pair:
the mapping of `(x, a)`. Notice also that if there were already a mapping from
`x`, we will have locally overwritten it. We call this behavior shadowing, and
the function extension in the lambda line shows how we implement that shadowing.

## Walking The Example

Now let us evaluate the big expression:

```racket
((lambda (i)
   ((lambda (j)
      ((lambda (j)
         i)
       5))
    6))
 7)
```

We evaluate the two sub-pieces. The `rator` evaluates to a procedure, and the
`rand` evaluates to `7`. When we do the invocation, we start to evaluate the
body:

```racket
((lambda (j)
   ((lambda (j)
      i)
    5))
 6)
```

We do so while keeping track of the association that `i` should mean `7`. If we
wrote the environment out by hand, it would now look like this:

```racket
(lambda (y)
  (if (eqv? y 'i)
      7
      ((lambda (y) <bomb>)
       y)))
```

So we evaluate that body in this environment. In the same fashion, we evaluate
the two sub-pieces, produce the procedure and the value `6`, and then do the
application. That means we evaluate this body:

```racket
((lambda (j)
   i)
 5)
```

with one additional binding, `j` to `6`. If we wrote the environment out by
hand, it would now look like this:

```racket
(lambda (y)
  (if (eqv? y 'j)
      6
      ((lambda (y)
         (if (eqv? y 'i)
             7
             ((lambda (y) <bomb>)
              y)))
       y)))
```

Each time we extend the environment, we wrap the old one with an `if` statement
that first checks for the variable we just added, and otherwise recurs to the
slightly smaller environment.

Doing the same again, we evaluate the two sub-pieces, do the Racket application,
and evaluate the body within the extended environment. In this case, that means
we evaluate `i` in the following environment:

```racket
(lambda (y)
  (if (eqv? y 'j)
      5
      ((lambda (y)
         (if (eqv? y 'j)
             6
             ((lambda (y)
                (if (eqv? y 'i)
                    7
                    ((lambda (y) <bomb>)
                     y)))
              y)))
       y)))
```

Notice two things here. First, as we focus more tightly on a sub-expression of a
sub-expression of a sub-expression, our environment has grown larger and larger.
There is an inverse relationship between how tightly focused we are on a
sub-expression and how large a nested function our environment has become.

Second, the innermost binding for `j` in our original expression, `5`, has now
blocked out the ability to look up the other meaning of `j`. This is our
implementation of shadowing. Because we accumulate these bindings with the most
recent one in front, we can use the recursion into the program's structure to
get the shadowing behavior right.

When we finally look up `i`, we check that it is not `j`, then that it is not
`j` again, and then finally that `'i` is `'i`. Since it is, the lookup returns
`7`, and the whole expression evaluates to `7`.
