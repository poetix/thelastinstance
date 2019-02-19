---
title: "Terminal Cases"
date: 2019-02-19T13:50:50Z
draft: true 
toc: false
images:
tags: 
  - computation
---
There is a misunderstanding of the _Entscheidungsproblem_ abroad which I should like to try to correct. This musunderstanding is expressed by statements such as "the only way to know whether a computer program will terminate is to run it". The real situation is both better and worse than this. It's worse because, in the general case, running a computer program is not an effective way to know whether it will terminate. At most we can determinate whether it will terminate given a certain allocation of resources (time, storage, electricity). But for any finite allocation of these things, it's always possible to devise a program that would terminate just after that allocation had been exhausted. The halting problem tells us that we can't write any program that will, with generality, correctly determine for every program fed to it as input whether that program will terminate. That includes a program that would simulate the execution of the input program up to a certain resource limit (which would be effectively the same as "running" the program).

But the situation is also better than "we can't ever tell whether _any_ program will terminate", or even, " we can't ever write a program that can tell whether _some_ programs will terminate". We can in fact reason about the behaviour of _many_ programs to determine whether or not they will terminate, and we can write programs that will perform the mechanical steps of some of this reasoning for us. We can even write a program that can determine, for a given input program, whether it describes a program that can be reasoned about in a particular way.

Take the following Haskell program. Does the function `toInt` defined here terminate?

```
data Nat = Zero | Succ Nat

fun toInt :: Nat -> Int
toInt Zero = 0
toInt (Succ n) = 1 + (toInt n)
```

We can see at a glance that it does, and the way we know that it does is through inductive reasoning about the data type `Nat`, the observation that the function `toInt` is _total_, and inductive reasoning about the recursive behaviour of the two cases `toInt` defines. Every `Nat` is either a `Zero`, for which `toInt` returns a constant result, or the successor to a `Nat`. If it's a successor, we compute `toInt` for the `Nat` it's the successor of, add 1 to the result and terminate, so if `toInt` terminates for any `Nat`, `toInt` also terminates for the successor to that `Nat`. Every `Nat` is thus either an input for which `toInt` terminates, or the successor to an input for which `toInt` terminates, for which `toInt` also terminates. There is no way to define a `Nat` that `toInt` won't be able to take apart and convert into an `Int` (let's say for the sake of argument here that `Int`s are unbounded and we can't hit an overflow exception. Exercise for the reader: write a version of this program that is guaranteed to return either an Int, or an `Overflow` value).

Now, it's possible to write a function checker that establishes whether a given function is total, and whether it recurses only in ways that are guaranteed to terminate for a total function. If both of these conditions are met, then we know that the function will terminate. What we have really done is determined whether it belongs to a _non-Turing-complete subset_ of our programming language. We haven't solved the halting-problem, because we haven't provided a way of determining the halting property of functions that don't belong to this subset. But we have carved away a part of the space of possible programs and proven the halting-properties of programs within that space.

What happens if we pass our function checker to itself? Is it possible to write a function checker as a total function?
