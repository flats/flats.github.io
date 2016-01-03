---
layout: post
title: "Closures, Yield, and the Enumerable Module"
date: 2015-12-10 18:08:24 -0500
comments: true
categories: flatiron ruby programming
---
I tried to make a class Enumerable by dynamically mixing in the Enumerable module and defining `#each` after declaring the class. I failed. This is my story.

- - -

I was messing around with dynamically mixing in Ruby modules. One of the most useful modules in Ruby is the `Enumerable` module. It's a total badass. So I thought I'd try mixing it into a class that I had already defined and see if that worked out.

The first step is including it dynamically. Maybe I can just call the include method on my class!
```ruby
MyClass.include(Enumerable)
```
Okay! That appears to work. No exceptions. Now let's use it. I'll try out using `#reduce`, one of my favorite higher-order functions in Ruby.
```ruby
my_thing = MyClass.new
my_thing.reduce { |thing| puts thing } # => NoMethodError: undefined method `each' for #<MyClass:0x007fe59c0e3588>
```
Oops. In order to take advantage of the Enumerable module, you need both to include the module and [define](http://www.rubydoc.info/stdlib/core/2.2.2/Enumerable) your own `#each` method.[^1] This makes a lot of sense - how can your class be enumerable in there's nothing to enumerate?

I could just define `#each` in my class definition and be done with it, but this is all about exploring the process of dynamically modifying classes. Let's just just pretend that doing that is not an option here and we want to add this instance method to the class after it's been defined.

Defining an instance method for a class after the class definition, however, is pretty awkward. We can't just use the dot syntax on our class, becuase that would definite our `#each` for just that one instance of `MyClass`. The best way to do this is to use `#define_method`. We can also use `Class#class_eval`, and it may be preferable from a [performance perspective](http://greyblake.com/blog/2012/09/02/ruby-perfomance-tricks/), but it's not very readable and unless we absolutely need to squeeze every ounce of performance out of this code, we should probably avoid it. Also, that's too easy. Let's make this harder. Let's use `#define_method`.

The problem here is that `#define_method` is a private method that every class inherits from `Module`. So we can't just call `MyClass.define_method`. We have to either create a method in the class definition that in turns calls `#define_method` for us,
```ruby
class MyClass
  def self.create_method(method_name, method_body)
    define_method(method_name, method_body)
  end
end

each_body = lambda {
  yield 1
  yield 2
  yield 3
}
MyClass.create_method(:each, each_body)
```
or we have to use `#__send__`.
```ruby
my_thing.class.send(:define_method, :each) {
  yield 1
  yield 2
  yield 3
}
```

All instances of `MyClass` will now have our `#each`. Great. Let's try using it now!
```ruby
my_thing = MyClass.new
my_thing.map { |thing| puts thing } # => no block given (yield) (LocalJumpError)
```
What? [LocalJumpError](http://ruby-doc.org/core-2.2.2/LocalJumpError.html)? No block given? I definitely just passed in a block. What went wrong?

### Block as Closures and the Behavior of `yield` ###

This is where we get into the hairy business of Ruby's closures. Closures are a bit of a strange concept, especially in the context of object-oriented programming, in which we rely on objects for pretty much everything. Still, it's worth knowing about closures, Ruby has them, and understanding them will help you in certain other functional or functional hybrid languages, such as JavaScript. Alan Skorkin has [a nice, succinct definition of closures on his blog](http://www.skorks.com/2010/05/closures-a-simple-explanation-using-ruby/):

> A closure is basically a function/method that has the following two properties:  
>  
> • You can pass it around like an object (to be called later)  
> • It remembers the values of all the variables that were in scope when the function was created. It is then able to access those variables when it is called even though they may no longer be in scope.

I recommend Mr. Skorkin's post about closures. It's clear and precise and covers both Ruby's implementation of closures and some common uses of closures. If closures are confusing to you, as they were to me, go ahead and read this post and come back when you're finished.

Ruby blocks are closures (or at least almost closures - see the second footnote). When you create a block and pass it to a method, it closes over all of the variables and constants in its lexical scope.Here's the crazy thing: Ruby blocks *also close over blocks*. They are insane cannibal constructs. So, the problem here is that the yield in the body of my `#each` is actually yielding to the implicit block (whether it's passed or not) passed to `#create_method`. Crazy pants.

As Paul Inning says in [his encyclopedic discussion of Ruby closures](https://innig.net/software/ruby/closures-in-ruby), "`yield` can *only* refer to the block passed to the method it's in."[^2] John Mair, creator of our beloved Pry REPL, [believes that this is actually pretty consistent behavior](https://banisterfiend.wordpress.com/2010/11/06/behavior-of-yield-in-define_method/), and that makes sense to me - because blocks close over blocks, any method can receive a block, and a method can handle only one block at a time, `yield` has to just refer to the block passed to the method in which it finds itself.

- - -

Phew. Okay, fine, I'll just use `Class#class_eval`. Or maybe I'll just actually `include Enumerable` and define `#each` like a normal human being.


[^1]: Also note that (from the Ruby documentation) "If `Enumerable#max`, `#min`, or `#sort` is used, the objects in the collection must also implement a meaningful `<=>` operator, as these methods rely on an ordering between members of the collection."

[^2]: He also says that this means that blocks aren't true closures - we can't pass them around. We can only send them to a method and use `yield` to invoke them. In other words, they're not [first-class functions](https://en.wikipedia.org/wiki/First-class_function). This is true.