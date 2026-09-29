---
url: "/2007/06/04/more-thoughts-on-language-support-of-tdd/"
title: "More Thoughts on Language Support of TDD"
date: 2007-06-04T14:21:47.000Z
slug: "more-thoughts-on-language-support-of-tdd"
tags: 
  - "tdd"
---

Thinking about [my earlier post](/2007/05/29/to-toop-or-oop-how-to-decide/ "To TOOP or OOP?  How to decide?") discussing the OOP vs TOOP problem, I mentioned at the end that the best solution to this problem in my mind would be integrated language support for test classes.  Specifically, a way to let the Compiler/Runtime know that a specific class is a test class, and should therefore be able to access any and every property of a class.

It occurred to me that such blatant intrusion into the privacy of a class is not unknown in the programming world.  C++ has the notion of a "Friend" class.  This is a class that can access all members of another class regardless of their protection level.  To keep things civil, so that just any class can't declare itself to be a Friend of any class it wants, the class that the Friend class would be accessing would declare specifically that classes X, Y and Z are fiends, and so can have free reign.  Granted this is considered to be rather scary, and one of those features that makes C++ an ideal tool for shooting ones own foot off.

But, this concept has some merit within the context of Test classes.  In the .Net world we could potentially use attributes on a class to identify what the test class (or classes I suppose) for a class are.  The compiler and runtime could then use that information to provide unlimited access only to the classes listed in the test class list.  For more protection perhaps it would also validate that the classes in the list also have the appropriate attribute (TestFixture in NUnit, TestClass in TFS) before allowing access.  You would need some additional tooling in the development environment so that you would get Intellisense on all of the private/protected members, but that should be easy for the MS guys after building in the test support, right?

There is some additional danger in this approach since there is some potential to modify IL at runtime, but couldn't there be additional protections around such things?  As someone with no knowledge of the internal workings of the CLR, I can't say for sure, but its worth trying.

So, I know officially propose to Microsoft that this feature be added to the 4.0 version of the framework.

Do you think they heard me?
