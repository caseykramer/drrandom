---
url: "/2008/04/08/how-important-is-your-code-audience/"
title: "How important is your Code Audience?"
date: 2008-04-08T17:30:04.000Z
slug: "how-important-is-your-code-audience"
tags: 
  - "uncategorized"
---

We had a friend visiting from out of town last night, and she is apparently fond of "Top Chef".  This was the first time I had encountered this particular "reality" based show, and I found in a lot of ways it was like the majority of the rest of the shows out there; trying hard to create drama where there really isn't any.  In this particular episode there was a challenge to cook food for a neighborhood block party.  The group that did the worst in this particular case cited the fact that they were preparing food for everyone including kids, as a reason for not getting jiggy with the menu (in a culinary sense).  The response from the judges was that good food would speak for itself, and they should have considered it their responsibility to raise the collective bar of quality for the neighborhood.

I find this quite interesting because it mirrors an internal dilemma I am facing with the code I am currently putting together.  The dilemma is this:  Should I create things which are more "advanced" and more difficult for those who are not familiar with the concepts, or should I take the audience into consideration, and forego some of the more advanced concepts in favor of simplicity and discoverability?

My gut tells me that I should do the latter.  I'm a big believer in the power of writing code that is easy to understand and maintain.  This is particularly true whenever you are writing code which will potentially be consumed by other developers working within the application.  If someone cannot figure out how to utilize the work you've done, then chances are they are going to go off and duplicate it.  The eventual result of this being half a dozen ways to accomplish what is basically the same thing.

The idea brought up on the show seems to go against this philosophy by suggesting that you have the ability to expand the pallets of those who are consuming your work (the food metaphor is staring to get pretty rich here), and so you would be doing them a disservice not to.  In my case, I've found places where my choice to keep things more like what everyone is used to has added complexity and ugliness elsewhere. 

So does this mean I should forge ahead and present concepts which are going to be new to others on the team, and potentially create code that will not be re-used as it should because it does not make sense to those needing to re-use it?  If I don't introduce these concepts am I missing an opportunity to bring up the overall quality of the code being by everyone?  Am I just struggling to fid answers to questions that I really shouldn't be asking, since the YAGNI police are keeping their eyes on things?
