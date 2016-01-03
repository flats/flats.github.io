---
layout: post
title: "Frozen Strings, Symbols, and Garbage Collection in Ruby 2.3/2.4"
date: 2016-01-03 03:21:09 -0500
comments: true
categories: 
---

I was perusing the paperclip gem's source code when I came upon the following line:

```ruby
code here
```

It's a very useful line when you're dealing with saving attachments to a filesystem (as long as there are fewer than a billion objects with attachments). But what first struck me was the preponderance of `#freeze` calls. What the hell is `#freeze` anyway?