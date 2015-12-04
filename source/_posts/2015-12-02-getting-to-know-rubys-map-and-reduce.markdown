---
layout: post
title: "Getting to Know Ruby's Map and Reduce"
date: 2015-12-02 21:27:10 -0500
comments: true
categories: "Flatiron School"
---
Map and reduce are most closely associated with functional programming. Even so, Ruby, the object-oriented-est of all object-oriented programming languages, has robust implementations of both of these extremely expressive higher-order functions. I'm going to explore a bit of what you can do with them.

First, the Basics
=================

As higher-order functions, each of these methods accepts a block and then calls it as it iterates over an enumerable.

Map (or collect)...

Reduce (or inject)... . Scheme, OCaml, and other functional programming languages refer to this as fold.

Some Interesting Map and Reduce Examples
========================================

In his series on functional programming in Ruby[^1], Nathan Kleyn mentions a question that Yehuda Katz asked several years ago about splitting a module path in a particular way. One solution proposed by Bradley Grzesiak[^2] demonstrates a particularly interesting use of reduce:

```
module_name = "X::Y::Z"
module_name.split('::').inject([]) { |memo,x| memo.unshift(memo.empty? ? x : "#{memo[0]}::#{x}") }
=> ["X::Y::Z", "X::Y", "X"]
```

Let's break this code down into separate steps.

1. Bradley splits `module_name`, which is our starting string, into the array ["X", "Y", "Z"].

2. He then uses the `inject` using the `inject(initial)` syntax, which will initialize the accumulator, called `memo`, to an empty array.

3. Now the iteration begins. If this new 'memo' array is empty, which will only happen on the first iteration of `inject`, `inject` adds that element ("X") to the beginning of `memo` (using `unshift`). Otherwise, `inject` adds a string to `memo` containing the result of the previous iteration ("X"), the separator ("::"), and the current element ("Y").

So, the final array is built backwards: ["X"], then ["X::Y", "X"], and then finally ["X::Y::Z", "X::Y", "X"]. Nifty.


[^1]: http://www.sitepoint.com/functional-programming-techniques-with-ruby-part-i/
[^2]: http://rubyquicktips.com/post/1018776470/embracing-functional-programming