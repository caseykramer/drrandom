---
url: "/2006/05/31/thread-exception-change-in-net-2-0/"
title: "Thread Exception Change in .Net 2.0"
date: 2006-05-31T15:47:26.000Z
slug: "thread-exception-change-in-net-2-0"
tags: 
  - "uncategorized"
---

I just learned that Microsoft changed the way unhandled exceptions in Threads are propogated in .Net 2.0 vs the way they worked in 1.1.  With 2.0 any unhandled exceptions in a thread will not only cause that thread to exit, but will also be bubbled up to the thread that started the offending thread.  Overall, I think this is a good thing, since not only will it force people to handle exceptions within their thread code, but it will also eliminate the problem of processes mysteriously stopping if there is an unhandled exception on a thread.  It will, however, cause some serious grief for any developers not expecting that sort of behavior.  I'm sure I'm not the only one who has seen less than steller multi-threaded programming tactics in applications which should be better behaived (heck, I've done my share of shoddy thread handling in the past).  
  
The one exception to this, of course, is the ThreadAbortException, which understandably, only effects the thread it is raised on.  Granted using Thread.Abort, or raising a ThreadAbortException is considered a brute-force approach, so it _shouldn't_ be happening anyway....
