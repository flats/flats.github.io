---
layout: post
title: "Closures, Yield, and the Enumerable Module"
date: 2015-12-10 18:08:24 -0500
comments: true
categories: 
---
I tried to make a class Enumerable by dynamically mixing in the Enumerable module and defining `#each`, and I failed. This is my story.

- - -

I was bored and I was messing around with mixing in modules. Yep, that's boredom. Anyway, one of the most useful modules in Ruby is the Enumerable module. It's a total badass. So I wanted to add the 

There are a couple of ways to define a class method outside of a class definition. The most obvious way to do it is to use dot syntax:
```ruby
def MyClass.my_method
  "Thanks for stopping by my_method."
end
```
What's actually happening here is that we're defining a singleton method for the class object called MyClass.

Alas, you can't use `#define_method` to define a class method - it can only define instance methods.

```

http://yehudakatz.com/2009/11/15/metaprogramming-in-ruby-its-all-about-the-self/

https://banisterfiend.wordpress.com/2010/11/06/behavior-of-yield-in-define_method/