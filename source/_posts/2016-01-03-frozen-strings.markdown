---
layout: post
title: "Frozen Strings, Symbols, and Garbage Collection in Ruby 2.3/2.4"
date: 2016-01-03 03:21:09 -0500
comments: true
categories: 
---

I was perusing [the paperclip gem's source code](https://github.com/thoughtbot/paperclip) when I came upon the following line (lib/paperclip/interpolations.rb, line 178):

```ruby
("%09d".freeze % id).scan(/\d{3}/).join("/".freeze)
```

It's a useful bit of code for saving attachments to a filesystem, but what struck me immediately was the preponderance of `#freeze` calls. What the hell is `#freeze` anyway?

Well, apparently `#freeze` is used pretty often by experienced Ruby developers. You'll see it especially in popular Gems that have gone through several versions and have a large number of contributors.

There are a couple of reasons you might want to use it. Here they are.

### Make an Object Immutable ###

Ruby constants are really just variables that you shouldn't change. Ruby will warn you if you change a constant, but it won't raise an exception. So, if you really want to have Ruby enforce the constancy of a constant, you can freeze that object.

```ruby
JEDI_MASTER = "Kenobi".freeze
JEDI_MASTER.prepend("Rey ") # => RuntimeError: can't modify frozen String
```

Remember that when you assign a frozen object to a variable, this does not prevent the variable from being reassigned to another object. The following code, for example, works just fine:

```ruby
JEDI_MASTER = "Kenobi".freeze
JEDI_MASTER = "Rey Kenobi"
```

You can extend this to create a whole class of objects that are constant from birth - just call `#freeze` at the end of a class's initialize function definition. This might come in handy if you're [trying to write Ruby in a functional style](http://blog.honeybadger.io/when-to-use-freeze-and-frozen-in-ruby/).

### Performance ###

As great as this is, this is not why paperclip uses `#freeze` three times in that line at the beginning of this post.

It's used for performance.

Well, #freeze simply makes a Ruby object immutable. It's useful here because there's no way to unfreeze an object in Ruby, which allows Ruby to make some optimizations that can significantly reduce a program's memory usage. Ruby no longer has to worry about your string being modified, so once you've frozen a string, any other frozen string that's created will simply be a reference to that first frozen string. This behavior is similar to that of Ruby's symbols in that one and only one frozen string object for any string value will exist in the lifetime of a program, with all subsequent frozen strings with that same value just being a reference to the original frozen string.

So why not just use a symbol? Well, this would make sense, but unfortunately, Ruby's symbol was not really properly garbage collected until Ruby ???, which means that a proliferation of symbols could have unfortunate repercussions for a program'm memory footprint. Frozen strings, then, are a backward-compatible way to reduce memory usage when you're dealing with non-dynamic stringa that you may end up creating time and time again (sometimes thousands of times!).

https://blog.blockscore.com/new-features-in-ruby-2-3/