---
url: "/2011/07/31/making-the-climb-chronicling-the-journey-from-c-to-scala/"
title: "Making the Climb: Chronicling the journey from C# to Scala"
date: 2011-07-31T19:41:00.000Z
slug: "making-the-climb-chronicling-the-journey-from-c-to-scala"
tags: 
  - "making-the-climb"
---

Hows that for a title, eh?  Yeah, I know, not great, but there is only so much creativity I can manage in a day, and as you will soon find out my mind is busy with all kinds of new and interesting thing, so any spare neural pathways which may have one been useful for something like clever titles are now just too busy to be bothered.

<!-- missing image: /image.axd?picture=2011/10/Photoxpress_17379649_2.jpg -->So what is this all about?  Several months ago I had an interesting experience.  One that I could almost compare to a religious experience, only not quite so….religious.  It started with the most excellent book [Seven Languages in Seven Weeks](http://pragprog.com/book/btlang/seven-languages-in-seven-weeks) by Bruce Tate, from our good friends at the [Pragmatic Programmers](http://pragprog.com/).  Now, I will admit that I’ve not actually read all of it…yet.  The reason is that while I found the first several languages (Ruby, Io, and Prolog) interesting, and challenging, it was [Scala](http://www.scala-lang.org) that had a certain something that kept me wanting to know more.  Mr. Tate actually didn’t care much for the syntax of Scala, and was rather pleased to move on to Erlang.  Meanwhile I was off and running downloading Eclipse and the Scala IDE, looking at testing frameworks, online documentations, and what’s this?  Android development with Scala!?!?  I was hooked.

### What is Scala all about?

> _2003 - A drunken Martin Odersky sees a Reese's Peanut Butter Cup ad featuring somebody's peanut butter getting on somebody else's chocolate and has an idea. He creates Scala, a language that unifies constructs from both object oriented and functional languages. This pisses off both groups and each promptly declares jihad.  
> [James Iry](http://james-iry.blogspot.com/) - [A Brief, Incomplete, and Mostly Wrong History of Programming Languages](http://james-iry.blogspot.com/2009/05/brief-incomplete-and-mostly-wrong.html)_

So to move on to the the elevator pitch on Scala.  Scala is a Hybrid Object-Oriented/Functional language build on the JVM which makes extensive use of type inference.  Now, a C#/.Net dev looking at this statement will most likely have a couple of responses.  Depending on how snarky the specific developer is the first will most likely be “So?”.  Followed by something like “So is F#, and C# has all that Functional stuff built-in…what’s the big deal?”.  Answering this question is one of the things I’m hoping to address…though maybe not directly. There are a lot of reasons I think Scala is interesting to a .Net developer, not the least of which is the idea that learning a new language can help open your eyes to new ideas,  patterns and idioms.  But also, there was a recent [announcement](http://www.scala-lang.org/node/10299) from the Scala CLR team that, thanks to lots of hard work, and some funding from Microsoft, the [Scala CLR compiler tools](http://www.scala-lang.org/node/168) are actually usable.  This means that we could have another language available on the .Net platform for which to play.

There are no doubt going to be a number of different posts where I address various topics in some order that is in no way thought out.  What I would like to try to do is to present a lot of the things that make the language interesting, and unique, and how it can be used to solve problems in ways that are not possible in C#.  In some cases I may draw comparisons to Java, as a means of explaining some specific nuance of the language, or just to put into perspective why Scala is an even more compelling option for Java developers.  For those wanting to play along at home, without going through the process of installing Scala, you can head over to [SimplyScala](http://www.simplyscala.com/) and try out their web-based Scala REPL.  When you install Scala, you also get a REPL that you can fire off from a command line (or, as I like to do, add it as a tab to [Console2](http://sourceforge.net/projects/console/))

#### A quick look at some key syntax differences…

Before I start digging in too deep, I want to get some of the syntactic differences out of the way, that way future posts can focus more on specific topics, without needing to take a side-track into syntax.  Scala has a very malleable syntax, which is a very interesting thing given it’s also a statically-types, compiled language.  We’re used to this sort of behavior from those unruly dynamic languages like Ruby, but from a compiled language?  Really?  How gauche…But lets start with the basics.  Scala is a language that uses curly braces, so it’s not terribly difficult for a C# dev to look at initially.  What is unusual is the way variables and methods (and properties for those methods) are declared.  The general pattern is `name`**:**`type`. I read somewhere that Martin Ordesky once said that the name is the most important thing when looking at declarations, and this pattern supports that.  So if you want to declare an _immutable_ variable (i.e. one that can not be reassigned) it would look like this:

##### Mutability

I’m going to gloss over the distinction between mutable, and immutable for the moment, but suffice to say, functional programming places a high value on  immutability, and Scala extends this idea to include a general preference for immutability when possible.

```
val age:Int = 6
```

Likewise a _mutable_variable declaration would look like this:

```
var age:Int = 6
```

This is not actually what you are likely to see in a "real" Scala program, however, since the Scala compiler makes extensive use of Type Inference. So in both of these cases the type can be left off, because there is plenty of information available to know what the types are:

```
val age = 6
```

When we're dealing with method or function definitions, then there are some limits to what can be inferred. Generally the types of parameters _can not_ be inferred, and the return type _can_ (though when declaring methods on a class, unless they are very simple, it's considered a good idea to specify the type. This is to avoid confusion for both you're reader and the compiler).  Functions are defined using the `def`keyword like this:

##### Function vs Method

The distinction between functions and methods is actually pretty simple.  A method exists on a Class (or Object, or Trait), whereas a function does not.  Because Scala is functional, you can declare functions outside of the scope of a class without causing the anger of the gods to come raining down upon you.

```
def printNumbers(nums:List[Int]) {
  nums.foreach(n => println(n))
}
```

There are a few interesting things here, but I have a feeling C# devs will know what is going on without a whole lot of guidance. Take it as given that the `println`is a method that prints something to standard out (cause it is...we'll talk about where it comes from later), and it's easy to see that this method translates to this in C#:

```
public void printNumbers(List<Int> nums)
{
    nums.ForEach(n => Console.WriteLine(n));
}
```

Lets look at some of the differences....In the Scala version, we're not specifying visibility. By default everything is pubic in Scala, which is a different world than we're used to in C#. Also, there is no `void` return type defined. In Scala there is actually a class defined that takes the place of `void` called `Unit`. The compiler knows we are dealing with a method returning `Unit` for a couple reasons, the first and most significant is that in Scala, the return value of a function is the value of the last statement in the function.  So the `foreach()` function on the `List` returns `Unit`, so therefore the `printNumbers` function also returns `Unit`. There is also the way the function is written...lets take a look at a function that returns a value:

```
def square(value:Int):Int = value * value
```

In this case there is an equal sign after the function definition, which indicates that the bits afterword are the body of the function, and that the result of the following block is the return value of the function. Also notice I left of the curly braces this time round. Since it was a simple single statement method, there is no reason to use them. As a matter of convention, if your creating a method that returns `Unit`, you leave off the equal sign, and use the curlies (as a matter of fact, if you leave off the equals, you _have_ to use the curlies).  If you need multiple statements, then you need to use the curlies.

So back to the `Unit` return type on the first method. We can also write out `printNumbers` function like this:

```
def printNumbers(nums:List[Int]) { nums.foreach(println(_)) }
```

Yes, I did throw another curve at you here...the `foreach` function on the `List` takes a function as an argument, and the format for lambda expressions in Scala is basically the same as C#, however you can skip input types on lambdas if there is only one, and use the underscore `_` instead. This looks similar to the way C# lets you pass in a reference to a method that matches a delegate signature, but is not actually the same thing. In the Scala case, the `println(_)`is still a lambda expression. So before I wrap things up for this first go round, I'm going to throw out another example of using the underscore. This has actually been used as fodder to argue that Scala syntax is overly obtuse and cryptic. These things are always subject to interpretation, and I tend to think that it really depends on the specific project and developers. If your organization is used to a particular pattern, then even if it seems obtuse from the outside, it is a well defined and established pattern...use it. Anyway, on to the example...here we're going to calculate the sum of a list of numbers:

```
def sum(nums:List[Int]) = nums.reduceLeft(_+_)
```

Ok, so lets start with a quick look at `reduceLeft`...this is a function that will apply another function to each element in a list, starting with the left-most, and accumulating the result along the way. So basically it looks at the list, applies the function to the first two elements, then takes that result and applies it to the third element, and so on. In this case we've defined our function as `_+_`, and if you recall the underscore is basically a stand-in for a lambda parameter. In this case, though, we have two params, but the Scala compiler is clever enough to figure out that we want to take the arguments in order, and apply the `+` method to them (yes, `+` is a method, not an operator...stick with me we'll talk about this down the line). This starts delving into a little bit of compiler magic, and so it makes sense that some folks would be uncomfortable with this. It starts to unveil some of the secrets that are hiding underneath the surface of a seemingly stuffy, curly-braced, JVM language. I'm going to leave it here for now, and let you chew on this for a bit. Next time, we're going to start talking about the Object-Oriented side of Scala. Stay tuned.
