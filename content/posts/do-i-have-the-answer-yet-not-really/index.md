---
url: "/2009/02/08/do-i-have-the-answer-yet-not-really/"
title: "Do I have the answer yet? Not really"
date: 2009-02-08T00:31:31.000Z
slug: "do-i-have-the-answer-yet-not-really"
tags: 
  - "practices"
---

So previously I posed a [question](/2008/04/08/how-important-is-your-code-audience/), which in it's simplest form is: Should you write code for the rest of your group (or at their proficency level), or should you write code as advanced as you need. and let it serve as an example for those on your team who are less advanced in their abilities.  The practical answer I have come up with is, like most answers of this type, "It depends".

I have decided to handle things this way:   
First, don't compromise.  If I feel something is a bad practice, or ultimately going to restrain future development, then do what needs to be done.  
But Also: Try to avoid introducing advanced concepts and idioms until there is a compelling example of what their benefit is.  This is probably something that applies more to working with existing apps, then greenfield development, but it certainly has bearing in both cases.  The big example for this that I ran into was IoC.  I am a big fan of IoC, but it is hard to come up with a good, concise explanation of why you need this additional tool.  I've been wanting to introduce IoC since I started working at [Envisage](http://www.envisagenow.com/), but the explanation "This will make things much easier later on" is not good enough...particularly when you are trying to embrace YAGNI.  So the key is to wait until you can actually demonstrate the advantage, and provide a before and after example of how things are done.

Lead by example, but make sure you have the examples.  To bring the food analogy back into play, it isn't good enough to create a gourmet meal, and tell the people who say they don't like it that, no, it really is better, and their just not sophisticated enough to know it.  It is better to start smaller, and build their appreciation up in increments.
