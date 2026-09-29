---
url: "/2007/01/02/capturing-programmer-intent/"
title: "Capturing Programmer Intent"
date: 2007-01-02T16:48:24.000Z
slug: "capturing-programmer-intent"
tags: 
  - "tdd"
---

I was listening to the [ArCast](http://www.skyscrapr.net/blogs/arcasts/default.aspx) [recorded](http://www.skyscrapr.net/blogs/arcasts/default.aspx?ID=595) with [Scott Hanselman](http://www.hanselman.com/blog/default.aspx) earlier today, and he was talking about the idea that [Non-Software artifacts should approach zero](http://www.hanselman.com/blog/ARCastnetInterviewedByRonJacobsAtTechEd2006.aspx).  If you've seen some of his posts, or listened to some [Hanselminutes](http://www.hanselminutes.com) podcasts, you have no doubt come across this idea before.  I like this particular phrasing mostly because it gets to the heart of what I think one of the most often overlooked aspect of the programming process is; Namely, the intent of the programmer.  
  
I think this is one of the most important things to take into consideration when looking at someone else's code (or even your code, more that about a week after you wrote it).  There are a lot of subtleties about a design which go away after the code is written and starts to gather dust.  If the developer had a particular design pattern in mind when they built a class structure, this information exists only in the mind of the developer, and maybe some technical spec doc which is lost in source control, or share point somewhere.  Someone else comming along may look at that class structure, and not see the scaffolding the original developer put there to support that pattern, and will most likely simplify the design, removing the pattern in the process.  
  
One proposed solution to this, which I saw in a [posting from a java developer](http://www.developer.com/tech/article.php/2106271) was to use marker interfaces to communicate this sort of intent.  That is, an interface which actually has no method declarations, but exists only to mark a specific class as being part of a pattern.  One additional advantage that the Java-Doc system allowed was the addition of the documentation around that interface into the docs generated for the implementing class.  This is not a bad idea, though it is terribly hard to enforce.  
  
I think [Windows Workflow](http://wf.netfx3.com/) will be a major contributor in this arena, allowing very explicit declarative syntax for creating code.  There may even be some potential to building WF activities around design patterns (hmmm...maybe I have a project).  This idea ties into all sorts of other areas of development, though. When designing web services the contract is what is used to communicate the developer intent, and therefore creating super methods that take DataSets or XML Blobs makes the contract basically useless.  .Net attribute-oriented programming also allows for this sort of thing, though I can't see it being flexible enough to serve as a declarative language extension (yet).   
  
Once again I think we have no choice but to look at Unit Tests as the single most effective way to communicate developer intent.  If the tests are named properly, and test coverage is high enough, we should be able to see all of the requirements, how the various components interact, and generally what the developer was thinking.  I've even written tests which assert a particular class implements an interface simply because I though that was a critical part of the design.  
  
What is my point?  Well, I guess its really just the beginning of a though process around how to better capture programmer intent.  What tools should there be?  We all know documentation doesn't work.  Unless your doing Model Driven Development, UML is usually as out of whack with the software as the documentation (or worse).  And I think everyone agrees that putting this information in a word document is the best way to ensure it does not end up in the final product.  
  
  

powered by [performancing firefox](http://performancing.com/firefox)
