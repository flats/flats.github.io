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

As higher-order functions, each of these methods accepts a function in the form of a block and then calls it as it iterates over an enumerable object.

`map` (or `collect`) returns an array containing the results of calling the supplied block on each element of the enumerable. In other words, map allows you to apply a function to every element of a data structure and receive the result.

`reduce` (or `inject`) returns a value that is the result of applying a binary operation to the return value of applying the supplied block to each element of the enumerable. Whoa. What a mouthful. In other words, `reduce` "reduces" each element of an enumerable to a single value, accumulates that value in a single variable, and then returns the value of the accumulator. Scheme, OCaml, and other functional programming languages refer to this as fold.

Some Interesting Map and Reduce Examples
========================================

In his [series on functional programming in Ruby](http://www.sitepoint.com/functional-programming-techniques-with-ruby-part-i/), Nathan Kleyn mentions a question that Yehuda Katz asked several years ago about splitting a module path in a particular way. [One solution proposed by Bradley Grzesiak](http://rubyquicktips.com/post/1018776470/embracing-functional-programming) demonstrates a particularly interesting use of reduce:

```
module_name = "X::Y::Z"
module_name.split('::').inject([]) { |memo,x| memo.unshift(memo.empty? ? x : "#{memo[0]}::#{x}") }
=> ["X::Y::Z", "X::Y", "X"]
```

Let's break this code down into separate steps.

1. Mr. Grzesiak splits `module_name`, which is our starting string, into the array ["X", "Y", "Z"].

2. He then uses the `inject` using the `inject(initial)` syntax, which will initialize the accumulator, called `memo`, to an empty array.

3. Now the iteration begins. If this new 'memo' array is empty, which will only happen on the first iteration of `inject`, `inject` adds that element ("X") to the beginning of `memo` (using `unshift`). Otherwise, `inject` adds a string to `memo` containing the result of the previous iteration ("X"), the separator ("::"), and the current element ("Y").

So, the final array is built backwards: ["X"], then ["X::Y", "X"], and then finally ["X::Y::Z", "X::Y", "X"]. Nifty.

Haitham Mohammad [put `transpose` through its paces](http://rubyquicktips.com/post/18842314838/some-array-magic-using-transpose-map-and-reduce) on [Ruby Quicktips](http://rubyquicktips.com/), the same site that gave us the above solution. Transpose, [according to the Ruby documentation](http://ruby-doc.org/core-2.2.0/Array.html#method-i-transpose), "assumes that self is an array of arrays and transposes the rows and columns." In other words, if you imagine an array of arrays as rows in a table, `transpose` will return the columns from that table.

- - -

Mr. Mohammad then shows us how to get the sum of each column of table represented by an array of row arrays.

```
a = [1, 2, 3]
b = [4, 5, 6]
c = [7, 8, 9]

[a, b, c].transpose.map { |x| x.reduce :+ }
# => [12, 15, 18]
```
