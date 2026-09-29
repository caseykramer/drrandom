---
url: "/2007/06/05/you-got-your-ddd-in-my-soa/"
title: "You got your DDD in my SOA!"
date: 2007-06-05T14:09:50.000Z
slug: "you-got-your-ddd-in-my-soa"
tags: 
  - "ddd"
---

I've been trying to catch up on past episodes of .Net Rocks while on my hour+ drive to work everyday, and this morning listened to the interview with [Jimmy Nilsson](http://www.jnsk.se/weblog/) ([episode # 191](http://dotnetrocks.com/default.aspx?showID=194 "DNR Episode 191")) on Domain Driven Design.  I'll admit now that I have not had time to read his book all the way through, but it is one of the ones that I'm most eager to dig into, since not only is he talking about DDD, but he's also talking about the [Martin Fowler](http://www.martinfowler.com) design patterns from [PoEEA](http://www.martinfowler.com/books.html#eaa "Patterns of Enterprise Application Architecture").

One of the things brought up on the show was [this post](http://www.jnsk.se/weblog/posts/SOA-QDDDTheOpposite.htm "SOA-Q: SOA the opposite to DDD?") on Jimmy's blog.  I was shocked at the idea that SOA and DDD could be considered opposite strategies by anyone.  One of the comments made by the DNR hosts was that SOA was a technology in search of a problem to solve, whereas DDD is focused on the problem domain to the exclusion of specific technologies.  This is the sort of brutal mischaracterization of SOA, that I think is still quite prevalent despite the efforts of folks like [Ron Jacobs](http://blogs.msdn.com/rjacobs/ "Ron Jacobs") who are trying to spread the word of SOA done right.  So here is my rundown of why, in fact, SOA and DDD go together like peanut butter and chocolate:

SOA is not a technology, it is a design strategy.  It is a way of thinking about your _business_ in terms of what services can be provided.  The idea that if your using SOA then you are automatically using Web Services is just plain wrong.  You can create a business application that is _Service Oriented_ without ever firing up the Web Service designer in .Net.  The core concept is to model the services used in your organization.  These services should in fact be recognizable objects from your Domain Model.  Business people should be able to talk about the services in the same language they use to talk about the same concepts within the business.  If that isn't happening, then you are not designing the right services.

The process of modeling services should be derived directly from the Domain Model created using DDD if those services are going to be useful to the business.  One of the arguments for going SOA is "Business Agility".  What does that mean in the real world?  Basically the ability to allow all of the aspects of your business to share information in a way that provides a measurable benefit.  The agility comes in when you are able to connect parts of your business in ways that were impossible before.  Once again, this has nothing to do with the technology behind SOA, it has to do with the business needs.  Granted from a technical standpoint, the easiest way to achieve the sort of connections we're talking about is to use a technology like web services, but that is not the _point_ of creating the SOA, and it certainly shouldn't be the overwhelming factor in deciding to use SOA.

So, in answer to Jimmy's question in his post (As I see it, there is lot of useful ideas there for SOA as well, or what am I missing?):  No, you aren't missing something, the SOA'ers (as you refer to them) are missing something.  You're right on.
