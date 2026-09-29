---
url: "/2011/10/05/making-the-climb-part-3-generics-of-a-higher-kind/"
title: "Making the Climb Part 3 - Generics of a Higher Kind"
date: 2011-10-05T16:24:00.000Z
slug: "making-the-climb-part-3-generics-of-a-higher-kind"
tags: 
  - "making-the-climb"
  - "scala"
---

This is part 3 in a series.  If you’ve not followed-along so far, you may want to check out [Part 1](/2011/07/31/making-the-climb-chronicling-the-journey-from-c-to-scala/) and [Part 2](/2011/08/14/making-the-climb-part-2-object-oriented-programming-in-scala/) first.

It’s time to start digging in to some of the crazy-goodness that makes Scala such a glorious and wonderful thing.  First things first, though, we have to talk a little bit about this history of Generics in Java and the JVM. 

#### Type Erasure and you…

Back around the time that .Net was adding support for generics, the folks in Java land were doing the same…sort of.  The biggest difference between the way Generics were implemented in .Net and Java is the fact that in .Net generics are supported directly in the CLR (sometimes called reified generics), whereas the JVM did not include direct support for generics.  Ok, so what does that actually _mean_?  Well, for starters it means that when you’re dealing with Generics in Java you run into [Type Erasure](http://download.oracle.com/javase/tutorial/java/generics/erasure.html).  Type Erasure means that when Java code with generics are compiled, the generic type information is removed at compile time, so while you’re looking at an ArrayList<String> in Java, the JVM sees this as just an ArrayList, and things get cast as needed.  There are a couple of types which get reified into actual types (Arrays are the best example), but for the most part this doesn’t happen.  In contrast in the .Net world a List<string> gets compiled into a List\`1<System.String> which is a real type at the IL level.  Now, I’m not going to get into the argument about whether or not type erasure is good or bad, but it is a fundamental difference between the two platforms. 

One very interesting effect of Type Erasure is that it allows you to specify a wildcard type argument for a generic type parameter.  In Java this looks something like ArrayList<?>, in Scala this same type would be ArrayList\[\_\].  Now, this is interesting because it allows you to side-step the whole issue of providing type arguments when you really don’t care what they are.  Now, lets back up one step real quick and talk about syntax for a moment. 

#### Getting down to business…

Those of you with sharp eyes will have noticed that Scala uses square brackets for type arguments.  You can limit types (in a way similar to generic constraints in C#) based on other types by using the following syntax: MyType\[T <: AnyRef\].  In this case we’re saying the Type argument T is a _subtype_ of AnyRef (which you may recall is the supertype for all classes).  This is known as a _Type Bound_, and specifically in this case an _[Upper Type Bound](http://www.scala-lang.org/node/136)_.  You could speculate (and would be correct to do so) that a [Lower Type Bound](http://www.scala-lang.org/node/137) works in the opposite direction, so MyType\[T >: A\] means T is a _supertype_ of A.  Now, this is interesting from a theoretical perspective, but there are practical uses for this when you bring in covariant and contravariant type parameters.  Scala allows you to denote a _covariant_ type like this: MyType\[+T\], and a _contravariant_ type like this: MyType\[-T\].  These are equivalent to the .Net 4 variance modifiers _in_ and _out_.  So the .Net version of our MyType declarations would be IMyType<in T> and IMyType<out T> respectively.  Note I added the _I_ to the type names since in .Net type variance declarations are limited to interfaces and methods.  This limitation does not exist in Scala, so you’re free to add variance modifiers wherever you like.  Ok, so why do I say type variance becomes interesting when you combine it with type bounds?  Well, lets look at some real live Scala code and see:

\[scala\]sealed abstract class Option\[+A\] extends Product { ... def getOrElse\[B &gt;: A\](default: =&gt; B): B = if (isEmpty) default else this.get ... }\[/scala\]

So this is a chunk of code from the Scala library. This is a sample of the [Option class](https://lampsvn.epfl.ch/trac/scala/browser/scala/tags/R_2_9_1_RC4/src//library/scala/Option.scala#L1), which is an implementation of the [Null Object Pattern](http://en.wikipedia.org/wiki/Null_Object_pattern), that is used everywhere within the Scala libraries. If you look at the type declaration you can see that Option takes a single type parameter `A` that happens to be covariant. This is because when you have an Option of a specific type you expect to be able to get that type back out of it. However, there is also this `getOrElse` method, which will either return the item, or return the value of the function you pass in (which should return the type `A`). Now if you were to try and make the argument to the function something like `=> A` you would get a compiler error because a covariant type is appearing in a contrvariant position. This happens in C# as well, if you were to do the same thing. So the solution to this is to give the `getOrElse` method a TypeParameter, and specify a lower bound of `A`, which means `B` has to be a supertype (or the same type as) `A`.  This is particularly awesome since we would like that to be a contravariant operation anyway.  This is a very nice trick, and one I wish it were possible to use in C#.

#### Types of Types of Types…..

But the goodness does not stop there.  We’re going to head off into the land where Scala really asserts itself in terms of it’s type system.  In Scala it is possible to specify that the Type Parameter of your class/method/whatever should itself also have a type parameter (which could also have a type parameter, that could have a type parameter….you get the idea).  This is known as Higher Kinded Types (from the idea of Higher Order Functions in Functional Programing, which are functions that take functions as arguments).  A [Kind](http://en.wikipedia.org/wiki/Kind_\(type_theory\)) in type theory is basically a higher-level abstraction over types which relates to types in basically the same way types relate to variables in terms of regular code (I know this didn’t actually help explain anything, but it sounds good right?).

You may be wondering why this is a big deal, or even what use it may have….well, lets turn to another example from the Scala library.  In this case we’re going to talk about the Map function that exists on pretty much every Scala collection (or collection-like) object.  This is a method that allows you to take a collection of one type, and turn it in to a collection of a different type…more or less equivalent to the Select() method from Linq on IEnumerable.  Now, what is really cool about the Scala Map method is that when you call it on a `List<y>` for example, and pass it a function to transform `y` into `x`,  you will get back a `List<x>`.  The important part here is the fact that it is a `List`…not an `Iterable`, not an Enumerable, not any other supertype in the heirarchy, but a `List`.  And to top it all off this method is defined at the top level of the Collections type hierarchy.  So if you create a new collection that inherits from one of the existing collection types, you get a Map method that behaves the right way free of charge.  This is impossible to do in C# without creating an implementation of the method for each object (and if you did that you would have no contract at the IList or IEnumerable level for the method).  I’ll let that sink in for a moment.  Now, there is a _lot_ of very cool things going on in the type system which will get their own posts (or twenty) at some point, as a matter of fact Scala's type system is actually Turing Complete in and of iteself, which makes my head hurt to thing about. But apart from that there is one more thing I want to share about Scala’s type parameters before wrapping this up.

#### A quacking all the way…..

In addition to specifying type arguments as parameters, you can actually get more granular and tell Scala you’ll take any object it’s got as a parameter, as long as it has some specific method declaration(s).  This is effectively duck typing, and makes testing really, really easy when you can use it.  The syntax for doing this looks like:

```
def MyMethod(thing: { def that(x:Int) }) = thing.that(2)
```

Take a long look at that for a sec, let it sync in.  Now, this is pretty good stuff by itself, but Scala’s type system gives us even more goodness.  One of the nifty things you can do is declare new types in terms of other types.  Something like aliasing….but with a lot more power, particularly when you take into account the fact that you can use the same “duck typing” syntax to define a type.  So here is what that would look like:

\[scala\]class Test { type Dog: { def bark; }

def itBarks(dog:Dog) = dog.bark }

class Wolf {

def bark = println("Actually, I tend to howl more")

}

class Poodle {

def bark = prinln("Le bark, Le bark") }

val test = new Test() test.itBarks(new Wolf()) test.itBarks(new Poodle())\[/scala\]

So, as you can see, you can use type definitions as a form of shorthand for the duck-typed references. What's even cooler is that you can actually define types at a package level as well, which means you can define them once in your project, and use them wherever you need them. This is something like the `using` aliases you can set up at the file level, only you can declare them at a much broader level. As a matter of fact, Scala has a special file called `[Predef](http://www.scala-lang.org/api/rc/scala/Predef$.html)` that gets run before anything else in a Scala program. This file defines a lot of nifty things, like the `println` method, including several types that can be used in any Scala program (check our the [source](https://lampsvn.epfl.ch/trac/scala/browser/scala/tags/R_2_9_1_RC4/src//library/scala/Predef.scala#L1) if you’d like). Pretty groovy, no?
