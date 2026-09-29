---
url: "/2007/01/16/more-on-marker-interfaces/"
title: "More on Marker Interfaces"
date: 2007-01-16T17:15:01.000Z
slug: "more-on-marker-interfaces"
tags: 
  - "uncategorized"
---

I've been reading the "Framework Design Guidelines: Conventions, Idioms, and Patterns for Reusable .NET Libraries" book recently, and I came across a section discussing interface design, which had direct bearing on one of my earlier posts regarding programmer intent.  They basically state flat out that you should never use marker interfaces in .Net.  Instead, you should favor custom attributes, and then test the type for that attribute.  This was interesting to me, since I have been trying to determine what, if any, value marker interfaces would have in .Net.  In the Java example I cited, one of the benefits was that the JavaDoc information associated with the interface would then be attached to the class, so you would have clear intent from the developer when the interfaces were used.  .Net documentation comments don't carry that same direct association...granted, when the documentation is generated, most of the time there will be a link to the interface definition..but it's not quite the same.  On the other hand, generally a custom attribute will not even provide that link, so from a doc standpoint there is less information available.   
  
I think, though, that it is a bit more important to have the information about developer intent available while reviewing the source code, which makes the custom attribute concept a really good option.  You could even conceivably create a utility which would grab the attribute information from the assembly meta-data and generate a report containing which classes were marked with which interfaces, and therefore, which classes were part of which patterns.  There would also be the ability to associate other information with the attribute, such as a description, so you could have an attribute which stated a class was part of a specific patter (say, Chain of Responsibility), and then give a description of the component.  With that you could see specific instances of Chain of Responsibility patterns within the code.  Attributes also follow inheritance hierarchy, so that could make things more confusing, depending on how the attribute was used on the more top-level classes.  
  
  

powered by [performancing firefox](http://performancing.com/firefox)
