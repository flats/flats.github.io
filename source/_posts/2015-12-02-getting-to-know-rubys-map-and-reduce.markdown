---
layout: post
title: "Getting to Know Ruby's Map and Reduce"
date: 2015-12-02 21:27:10 -0500
comments: true
categories: "Flatiron School"
---
Map and reduce are most closely associated with functional programming.[^1] Even so, Ruby, the object-oriented-est of all object-oriented programming languages, has robust implementations of both of these extremely expressive higher-order functions. I'm going to explore how to use these methods and what you can do with them, especially when you use them together. This will be pretty exhaustive, so be prepared for a *very* long blog post.

First, the Basics
=================

As higher-order functions, each of these methods accepts a function in the form of a block. They then iterate over an enumerable object and call this block on each iteration. Where they differ is in what they do with the result of this block.

`map` (or `collect`) returns an array containing the results of calling the supplied block on each element of the enumerable. In other words, map allows you to apply a function to every element of a data structure and receive the result.

`reduce` (or `inject`) returns a value that is the result of applying a binary operation to the return value of applying the supplied block to each element of the enumerable. Whoa. What a mouthful. In other words, `reduce` "reduces" each element of an enumerable to a single value, accumulates that value in a single variable, and then returns the value of the accumulator. Some functional languages, such Scheme and OCaml, refer to this as `fold`.

The simplest use of `reduce` is just `collection.reduce(initial_value, :name_of_method)`. For example,

```
array = [1, 2, 3, 4] # => [1, 2, 3, 4]
array.reduce(0, :+) => 10
```

calls the `:+` for each element in the array. The method that we specify (in this case, `:+`) has to be a binary method (a method called on one object with a single argument of another object), so we need a starting value, which we have in this case specified as 0. As we iterate through the array using reduce, this is what happens at each stage of the iteration:

```
# reduce sets up an accumulator variable, memo, and sets it to the initial
# value, which is 0.
memo = 0
# Now we hit the first element of the array.
memo.send(:+, 1) # => 1
# The line above is the same as memo += 1
memo.send(:+, 2) # => 3
memo.send(:+, 3) # => 6
# This is the last time through the array, so the value of memo at the end
# of this is the return value for reduce.
memo.send(:+, 4) # => 10
```

If you don't declare an initial value, the initial value will be the first element of the collection. There's also the option of passing a block instead of the name of a function, which would look like

```
array.reduce(0) { |sum, element| sum + element } # => 10
```

for the example above.

Some Interesting Map and Reduce Examples
========================================

These functions are very powerful when used together. First, I'll look at a cool example of using `map` on its own, but then I'll get into some uses of reduce chained on to the end of map.

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

- - -

Haitham Mohammad put `transpose` [through its paces](http://rubyquicktips.com/post/18842314838/some-array-magic-using-transpose-map-and-reduce) on [Ruby Quicktips](http://rubyquicktips.com/), the same site that gave us the above solution. `transpose`, [according to the Ruby documentation](http://ruby-doc.org/core-2.2.0/Array.html#method-i-transpose), "assumes that self is an array of arrays and transposes the rows and columns." In other words, if you imagine an array of arrays as rows in a table, `transpose` will return the columns from that table.

Mr. Mohammad then shows us how to get the sum of each column of table represented by an array of row arrays.

```
a = [1, 2, 3]
b = [4, 5, 6]
c = [7, 8, 9]

[a, b, c].transpose.map { |x| x.reduce :+ }
# => [12, 15, 18]
```

- - -

Finally, I'll use these two functions with lambdas in an example that I've whipped up for this blog entry.[^2] Suppose we have a hash of restaurant reviews that look like this:

```
{
    name: "A Nice Restaurant",
    address: "123 Nice Pl.",
    city: "Nice City",
    reviews: {
        {
            reviewer_name => 'John Doe',
            reviewer_score => 4,
            reviewer_would_recommend => true,
            review_blurb => "I love this place!"
        },
        {
            reviewer_name => 'Jane Doe',
            reviewer_score => 5,
            reviewer_would_recommend => true,
            review_blurb => "I love, love, love this place!"
        },
        {
            reviewer_name => 'Dohn Joe',
            reviewer_score => 2,
            reviewer_would_recommend => false,
            review_blurb => "I do not like this place."
        },
        {
            reviewer_name => 'Dane Joe',
            reviewer_score => 1,
            reviewer_would_recommend => false,
            review_blurb => "I hate this place!"
        },
        {
            reviewer_name => 'John Jane',
            reviewer_score => 1,
            reviewer_would_recommend => false,
            review_blurb => "Do not go to this place."
        }
    }
}
```

Obviously, this is a very simplistic representation of how you would actually design a data structure for restaurant reviews, but it'll do for our purposes.


[^1]: [Lisp](https://en.wikipedia.org/wiki/Lisp_(programming_language)), one of the oldest high-level programming languages, pioneered the use of higher-order functions. While it supports several different programming paradigms, Lisp and its most popular dialects, Common Lisp and Scheme, are most commonly used to program in the functional programming paradigm.

[^2]: Credit to [David Yeung's blog](http://yeungda.com/2011/11/01/ruby-lambda-keyword.html) for examples of using higher-order functions with lambdas.