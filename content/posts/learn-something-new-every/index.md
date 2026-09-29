---
url: "/2006/05/24/learn-something-new-every/"
title: "Learn something new every...."
date: 2006-05-24T16:30:23.000Z
slug: "learn-something-new-every"
tags: 
  - "uncategorized"
---

It just goes to show that there are always little surprises waiting around the corner.  I've been looking at least casually at .Net 2.0 since Beta 2, and I even read through the "What's New" for C# 2.0 to see if I missed anything.  Somehow I managed not to notice Implicit Delegate Assignment (or at least that's what I'm calling it).  
  
It is now possible to assign a delegate using just the Class and Method name...you no longer have to create a new instance of the delegate type.  Not a huge thing, I know, but for those of us who are easily impressed it's, well, impressive.  
  
Here's the skinny in code language:  
  

```
public class SomeClass
{
    public event EventHandler MyEvent;
}

public class SomeOtherClass
{
    SomeClass _class; 
    public SomeOtherClass()
    {
        _class = new SomeClass();   
        // Old Way
        _class.MyEvent += new EventHandler(this.MyHandler);
        
       // New Way
       _class.MyEvent += this.MyHandler;
    }

    public void EventHandler(object sender, EventArgs args)
    {
        // Handler Code
    }
}

```
