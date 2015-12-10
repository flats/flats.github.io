---
layout: post
title: "Getting to Know Ruby's Map and Reduce"
date: 2015-12-02 21:27:10 -0500
comments: true
categories: "Flatiron School"
---
The `#map` and `#reduce` functions are most closely associated with functional programming.[^1] Even so, Ruby, the object-oriented-est of all object-oriented programming languages, has robust implementations of both of these extremely expressive higher-order functions. I'm going to explore how to use these methods and what you can do with them, especially when you use them together. This will be pretty exhaustive, so be prepared for a *very* long blog post.

First, the Basics
=================

As [higher-order functions](https://en.wikipedia.org/wiki/Higher-order_function), each of these methods accepts a function in the form of a block. They then iterate over an enumerable object and call this block on each iteration. Where they differ is in what they do with the result of this block.

`#map` (or `#collect`) returns an array containing the results of calling the supplied block on each element of the enumerable. In other words, map allows you to apply a function to every element of a data structure and receive the result.

`#map` is usually called with a block, as in `collection.map { |element| element + 1 }`. If `collection = [1, 2, 3, 4]`, this will return `[2, 3, 4, 5]`. As with many other methods from the Enumerator module, if `#map` is called without a block, it will return an Enumerator.[^2]

`#reduce` (or `#inject`) returns a value that is the result of applying a binary operation to the return value of applying the supplied block to each element of the enumerable. Whoa. What a mouthful. In other words, `#reduce` "reduces" each element of an enumerable to a single value, accumulates that value in a single variable, and then returns the value of the accumulator. Some functional languages, such Scheme and OCaml, refer to this function as `fold`.

The simplest use of `#reduce` is just `collection.reduce(initial_value, :name_of_method)`. For example,
```ruby
array = [1, 2, 3, 4] # => [1, 2, 3, 4]
array.reduce(0, :+) => 10
```
calls the `#+` for each element in the array. The method that we specify (in this case, `#+`) has to be a binary method (a method called on one object with a single argument of another object), so we need a starting value, which we have in this case specified as 0. As we iterate through the array using reduce, this is what happens at each stage of the iteration:
```ruby
# reduce sets up an accumulator variable, memo, and sets it to the initial
# value, which is 0.
memo = 0
# Now we hit the first element of the array.
memo.send(:+, 1) # => 1
# The line above is the same as memo += 1 or memo.+(1).
memo.send(:+, 2) # => 3
memo.send(:+, 3) # => 6
# This is the last time through the array, so the value of memo at the end
# of this is the return value for reduce.
memo.send(:+, 4) # => 10
```
If you don't declare an initial value, the initial value will be the first element of the collection. There's also the option of passing a block instead of the name of a function, which would look like
```ruby
array.reduce(0) { |sum, element| sum + element } # => 10
```
for the example above.

Some Interesting Map and Reduce Examples
========================================

These functions are very powerful when used together. First, I'll look at a couple of examples of using `#map` and `#reduce` on their own, and then I'll get into an example of using reduce chained on to the end of map.

### Reduce as a Recursion Alternative ###

In his [series on functional programming in Ruby](http://www.sitepoint.com/functional-programming-techniques-with-ruby-part-i/), Nathan Kleyn mentions a question that Yehuda Katz asked several years ago about splitting a module name in a particular way.[^3] [One solution proposed by Bradley Grzesiak](http://rubyquicktips.com/post/1018776470/embracing-functional-programming) demonstrates a particularly interesting use of `#reduce` (or, as in this case, `#inject`:
```ruby
module_name = "X::Y::Z"
module_name.split('::').inject([]) { |memo,x| memo.unshift(memo.empty? ? x : "#{memo[0]}::#{x}") }
=> ["X::Y::Z", "X::Y", "X"]
```
Let's break this not-quite-recursive code down into separate steps.

1. Mr. Grzesiak splits `module_name`, which is our starting string, into the array ["X", "Y", "Z"].

2. He then uses `#inject` with the `inject(initial)` syntax, which will initialize the accumulator, called `memo`, to an empty array.

3. Now the iteration begins. If this new 'memo' array is empty, which will only happen on the first iteration of `#inject`, `#inject` adds that element ("X") to the beginning of `memo` (using `Array#unshift`). Otherwise, `#inject` adds a string to `memo` containing the result of the previous iteration ("X"), the separator ("::"), and the current element ("Y").

So, the final array is built backwards: ["X"], then ["X::Y", "X"], and then finally ["X::Y::Z", "X::Y", "X"]. Nifty. There is certainly a recursive solution to this problem, but this is a great example of how higher-order functions can be used as an alternative to recursion.

### Map Arrays of Strings ###

The following is a chunk of code from the [MailHelper module of Rails 4.2.5's ActionMailer module](https://github.com/rails/rails/blob/master/actionmailer/lib/action_mailer/mail_helper.rb).
```ruby
# Returns +text+ wrapped at +len+ columns and indented +indent+ spaces.
# By default column length +len+ equals 72 characters and indent
# +indent+ equal two spaces.
#
#   my_text = 'Here is a sample text with more than 40 characters'
#
#   format_paragraph(my_text, 25, 4)
#   # => "    Here is a sample text with\n    more than 40 characters"
def format_paragraph(text, len = 72, indent = 2)
  sentences = [[]]

  text.split.each do |word|
    if sentences.first.present? && (sentences.last + [word]).join(' ').length > len
      sentences << [word]
    else
      sentences.last << word
    end
  end

  indentation = " " * indent
  sentences.map! { |sentence|
    "#{indentation}#{sentence.join(' ')}"
  }.join "\n"
end
```
There's nothing too remarkable about this method and its use of map, but I'm going to break it down because it's a good example of `#map` in some very widely used code. This code is proobably executing somewhere at this very moment.

As the comment says, this method `format_paragraph` takes a chunk of text that needs formatting, a line length that defaults to 72, and an indent length (in spaces) that defatuls to 2. There are two steps to this process: first break the text into lines, and then indent each line by the specified indent size.

In the first step, our function creates an array of arrays `sentences`, each array of which comprises the strings that make up each line. The combined character count of the words in each of these nested arrays is not greater than the specified line length (`len`).

The second step is where we use map to add the proper indentation to each line. After building up the indentation as a string of whitespace using `String#*`, our function mutates the array of line arrays by using `#map!` to join the words with a space (rebuild then sentence) and prepend the indentation spaces. Then the lines are all joined with line breaks. Now we have a formatted paragraph.

### Map and Reduce in Tandem ###

Haitham Mohammad put `#transpose` [through its paces](http://rubyquicktips.com/post/18842314838/some-array-magic-using-transpose-map-and-reduce) on [Ruby Quicktips](http://rubyquicktips.com/), the same site that gave us the above solution. `#transpose`, [according to the Ruby documentation](http://ruby-doc.org/core-2.2.0/Array.html#method-i-transpose), "assumes that self is an array of arrays and transposes the rows and columns." In other words, if you imagine an array of arrays as rows in a table, `#transpose` will return the columns from that table.

Mr. Mohammad then shows us how to get the sum of each column of table represented by an array of row arrays.
```ruby
a = [1, 2, 3]
b = [4, 5, 6]
c = [7, 8, 9]

[a, b, c].transpose.map { |x| x.reduce :+ }
# => [12, 15, 18]
```
- - -

So there's a bit about `#map` and `#reduce`. There's a lot more to discuss - for example, these methods [can take lambdas, too!](http://yeungda.com/2011/11/01/ruby-lambda-keyword.html) - but this is already a very long post.

- - -

I love these guys.

[^1]: [Lisp](https://en.wikipedia.org/wiki/Lisp_(programming_language)), one of the oldest high-level programming languages, pioneered the use of higher-order functions. While it supports several different programming paradigms, Lisp and its most popular dialects, Common Lisp and Scheme, are most commonly used to program in the functional programming paradigm.

[^2]: Enumerators are a large topic in and of themselves - for more information, see the [Ruby documentation](http://ruby-doc.org/core-2.2.0/Enumerator.html).

[^3]: I'm curious as to why wycats was working on this. Obviously this has something to do with breaking down the parsing of a class or module, but why? If anyone has some insight into context for this problem, [let me know](mailto:david.flaherty@flatironschool.com).