---
url: "/2007/07/13/taking-the-coderush-plunge/"
title: "Taking the CodeRush Plunge"
date: 2007-07-13T15:00:21.000Z
slug: "taking-the-coderush-plunge"
tags: 
  - "tools"
---

I recently decided to take the plunge and get [CodeRush](http://www.devexpress.com/Products/NET/IDETools/CodeRush/Index.xml) and [Refactor! Pro](http://www.devexpress.com/Products/NET/IDETools/Refactor/Index.xml) (along with [DxCore](http://www.devexpress.com/Products/NET/IDETools/DXCore/Index.xml)) loaded instead of [Resharper](http://www.jetbrains.com/resharper/).  Now, don't get me wrong, there is a lot I like about Resharper, but overall the performance was becoming an issue.  There were often problems with VS freezing for no particular reason, and then coming back as if nothing was wrong...I swear it was like my IDE had narcolepsy or something.

One of the things I noticed immediately about CodeRush was the fact that there was a single installer, and when I went to run it I was able to install it on all versions of Visual Studio, including the Orcas Beta.  This was nice when compared to Resharper's separate install for vs 2005 and vs 2003.  It also makes me feel good about improvements in the product being available for all versions of the IDE.  One thing that I noticed about R# was that there was some work being done in the VS 2005 version around performance, but that did not seem to trickle down to the VS 2003 version.  I think the big reason for this is the fact that CodeRush and Refactor! are implemented on top of DxCore, which provides a very clean abstraction from the scariness that is the Visual Studio integration layer.

Here are the things I really like about CodeRush/Refactor:  

- The visualizations are stunning!  No, seriously this is some amazing stuff.  Circles, arrows, animations, eye candy yes, but useful eye candy.
- The Refactoring Live Preview is crazy brilliant.  [Mark Miller](http://www.doitwith.net/) mentioned this on a [DNR episode](http://www.dotnetrocks.com/default.aspx?showNum=185), and I agree with his comment that the live preview allows you to discover new useful refactorings that may not be completely obvious from the name.
- Performance is great.  'Nuf said.
- It works with VB.Net.  Granted I don't use VB.Net, but some of my co-workers do, and occasionally I have to work on VB.Net projects.
- Dynamic Templates rock.  The fact that I can create a new mnemonic for my type, and then use predefined prefixes and do vee<space> to create a new Employee Entity for example is pure bliss.
- Template contexts are way cool.  By default they have NUnit templates defined, and with contexts, t<space> creates both a TestFixture class and a Test method
- Markers and navigation are dreamy.
- There are some crazy-cool template functions, like the ability to do a foreach within a template, so things like creating a switch/case statement for all items in an enum can be done easily.  This also powers a conditional to case refactoring that is pretty sweet.

Here are the things that I miss from Resharper:

- Automatically adding a using statement in VS 2003 was sweet.  VS 2005 can do it with the buit-in intellisense features, but I got very used to it.
- The VS 2003 test runner was very nice.  I use the [Testdriven.Net](http://www.testdriven.net/) plugin, which I cannot live without, but I like the graphical runner in the IDE.  The [free test runner](http://www.jetbrains.com/unitrun/) from JetBrains is for VS 2005 only, so it doesn't help those of us in VS 2003.  I do like the fact that they released it as a free tool, though.
- The "Extract Field" refactoring doesn't exist in Refactor!...This shocked me a lot.
- The Find Usages task.  This I think is part of the reason why R# was slow, but it did a brilliant job.  I think the rename in R# was more powerful as well.  I think there is a rename in Beta for Refactor! that is supposed to be able to work accross an entire project/solution, but I haven't had a chance to really test it yet.
- The pre-build error checking is nice.

The good news is that the DxCore extensibility model means that most if not all of these items could be recreated.  The bad news is that there isn't a lot of documentation around the extensibility model, particularly when it comes to creating new refactorings.  The test runner is one of the most painful points for me right now, so I've started exploring the process of creating one using the DxCore APIs.  It opens up the possibility of refining things too, which would be nice.  What I would really like would be the ability to detect and integrate with TestDriven.Net.

The folks over at [Eleutian](http://blog.eleutian.com/) are evidently [running both](http://blog.eleutian.com/2007/01/28/ProductivityToolsPart1.aspx), which they claim is possible with some tweaking, but the performance issues for VS 2003 are the biggest downer on the R# side, so I will probably not go down that path.  I may load up the test runner for VS 2005, unless I get some time to try and build one using DxCore, in which case I'll share it with the rest of the world.
