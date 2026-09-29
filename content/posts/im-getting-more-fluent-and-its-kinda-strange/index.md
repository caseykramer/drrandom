---
url: "/2007/09/15/im-getting-more-fluent-and-its-kinda-strange/"
title: "I'm getting more fluent...and its kinda strange"
date: 2007-09-15T15:29:40.000Z
slug: "im-getting-more-fluent-and-its-kinda-strange"
tags: 
  - "uncategorized"
---

About two weeks ago I crafted my first Fluent Interface.  Since then I'm finding myself seeing more and more places where I think such an approach would be useful.  The part that I'm finding odd is that it is something that just recently became a possibility for me.  The big motivating factor behind that I believe was reading the [post](http://martinfowler.com/bliki/FluentInterface.html) that [Martin Fowler](http://martinfowler.com/bliki) did on the subject, in which he basically describes it as a super-great idea (okay, I'm paraphrasing, but you get the idea).  The concept is fairly simple; write an API that reads like a sentence. 

This isn't a new concept, in fact I seem to recall reading quite a bit in the world of agile and TDD in which the authors encourage you to make method/property/variable names verbose and more like natural language in order to improve readability of code, and make the items more self-documenting.  I think the big difference, though, is that fluent interfaces tend to be more granular.  Instead of a single method that reads like a sentence, you are building a sentence using method and property names, with Intellisense there to help you determine what is possible at the end.

The big shift, I think, is in the realization that within this context method names like With, For, and And are perfectly okay, and as a matter of fact make things better in the end.  Its like a taboo has been lifted, and suddenly I have a whole new landscape of possibilities opened up.

Since the original implementation of a small fluent interface I created for a small part of my project (I'm using it to describe discrete elements of a document to be parsed), I've found myself creating a new fluent interface to play with .Net 3.5 extension methods (replicating the Ruby 10.Minutes.Ago semantics), and also adding a new interface to the same production project as the first which is being used to grab component services from my IoC container.

I'm not sure if this is a new paradigm, or just a new hammer looking for nails, but it is interesting none-the-less. It has also opened up new challenges around testing and intellisense documentation, which I've not quite figured out yet.
