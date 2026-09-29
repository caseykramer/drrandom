---
url: "/2012/01/22/making-the-climb-part-4-pattern-matching/"
title: "Making the Climb Part 4–Pattern Matching"
date: 2012-01-22T20:23:58.000Z
slug: "making-the-climb-part-4-pattern-matching"
tags: 
  - "functional-programming"
  - "making-the-climb"
  - "scala"
---

Continuing our journey down the path from the familiar to the down-right bizarre, we find ourselves at Pattern Matching.  This is a feature of the Scala language that shows it’s functional side in a strong way.  Pattern Matching is a fundamental part of functional languages in general, and provides a way to write very concise and expressive code.  On the surface, pattern matching in Scala looks an awful lot like switch statements in C# (and Java for that matter), but you shouldn’t cling too hard to that association.

#### The Basics

Let’s start with a basic example that mimics the behavior of a switch statement.  Here is a quick sample entered directly into the Scala interpreter: \[scala\]scala> val x = "Hi" x: java.lang.String = Hi scala> x match { | case "Hi" => "Hi yourself" | case "Ho" => "Am Not!" | case \_ => "What?" | } res1: java.lang.String = Hi yourself\[/scala\] Yes, this looks really boring…but the syntax should be clear. The Pattern Match is invoked using the `match` keyword, followed by a block containing `case` expressions. It's worth pointing out at this point that the match is an expression, which means it has a value when evaluated (which is why the result from the interpreter is a string, and we don't have/need any `return` statements). This means that the results of the match can be assigned to a variable, or used as the return value of a function. The case statements in this example are simple, they match against string literals. The exception being the last, which uses the special wildcard match expression `_`.  As may be expected, evaluation happens in order, and the first expression which contains a matching pattern is the one that is executed.  Had the previous example placed the wildcard pattern first, then that would be evaluated every time, regardless of what value is passed in.

Now if this is all you could do with Pattern Matching, it wouldn’t be all that interesting, and I probably wouldn’t be sharing it with you.  The cool thing about Pattern Matching expressions is that you are **not** limited to literals or enum values like you are with C#.  One of the things you can do is match based on type: \[scala\]x match { case x:String => "You gave me a string: "+x case x:Int => "You gave me a number: "+x.toString case x:MyThing => "You gave me some thing: "+x.toStirng\[/scala\] A contrived example to be sure, but as you can see it is really easy to handle different types using the standard Scala type notations.  The return value of the match expression will be the most specific type that is the result of all possible expressions.  You need to be a little careful with this, since if a match expression appears at the end of a function, then the function’s return value will be the same as the match expressions.

#### Deconstruction

Now, type based matching is pretty cool, and honestly I’ve wished for this kind of functionality in C# before, but there is more.  One of the primary uses for Pattern Matching in purely functional is for _deconstruction_ of data structures.  By this I mean extracting values from data structures so you can use the data elements directly.  A quick example using a basic tuple would look like: \[scala\]x match { case (y,z) => "A tuple with "+y+" and "+z case \_ => "Something else" }\[/scala\] If `x` is a tuple, then this expression will print the values of both elements. The pattern (in this case `(y,z)`) binds the variable `y` to the first element in the tuple, and `z` to the second. If, for example, you didn't care about the value of the second element in the tuple, then you could use the wildcard character in the pattern: \[scala\]x match { case (y,\_) => "The first element from the tuple is "+y case \_ => "Something else" }\[/scala\] A common use for Pattern Matching in functional languages is to split a list into it's head (fist element) and tail (every other element). You can do this in Scala with a pattern that looks like: \[scala\]list match { case Nil => "Empty list" case x :: Nil => "Single element list: "+x case x :: xs => "List has a head of "+x +" and tail: "+xs.map(\_.toString).reduce(\_ + ","+ \_) case \_ => "Not a list" }\[/scala\] Looking at the patterns here, we have some interesting options. The first matches agains `Nil`, which is an empty list. The second uses the pattern `x :: Nil`, which is a list with a single element (and binds that element to `x`). The next pattern `x :: xs` divides the list into head (bound to `x`) and tail (bound to `xs`) segments. These are the standard three types of matches you see when pattern matching against lists.

#### Extractors

This ability to deconstruct objects is not limited to standard built-in types.  Scala has a generalized pattern called Extractor Objects which provide a way to create objects that can be used in Pattern Matching.  Lets put together another cheesy example to demonstrate this: \[scala\]class MyThing(val one:Int, val two:String)

object MyThing { def apply(thing1:Int,thing2:String) = { new MyThing(thing1,thing2) } def unapply(x:MyThing):Option\[(Int,String)\] = { Some(x.one,x.two) } }

val m = MyThing(2,"one")

m match { case MyThing(y,z) => "MyThing with two items: "+y+" and "+z case \_ => "Not a MyThing" }\[/scala\] For sake of completeness I've included the `apply` method, which (you may recall) is how you create `object`s in Scala. It also provides some context for the `unapply` method so things are a little less confusing  Since Pattern Matching performs deconstruction, it seems only logical that the method that does this work is called `unapply`. This method may look a little funny, but basically this is what happens. The item you are doing the `match` against is passed into the `unapply` method. If the method returns a `Some` value, then that is the expression that is evaluated, otherwise it moves on to the next. Also, if the item doesn't match the type of the argument to the `unapply` method, then it will skip that expression. If you're `unapply` returns a `Some[x]` then that value gets bound to the variables. In this case we have two, so we're returning them in a tuple.

#### Case Classes

Now, this is cool, but there is a lot of typing involved. Scala has a handy-dandy short-cut for doing this sort of thing called _Case Classes_. A Case Class allows you to define a basic class, and it automatically adds the companion `object` type with `apply` and `unapply` methods, along with accessesors for any constructor arguments. So, using Case Classes we can rewrite the previous example as: \[scala\]case class MyThing(one:Int, two:String)

val m = MyThing(2,"one")

m match { case MyThing(y,z) => "MyThing with two items: "+y+" and "+z case \_ => "Not a MyThing" }\[/scala\] As you can see, this removed the need for the companion object all together. Granted, all of the code is still there after the magic from the compiler, but there is way less typing involved.

#### Guards

As if Pattern Matching wasn't cool enough, you can further refine results from the match using _Guards_. This basically gives you a way to add additional conditions to a pattern using standard expressions which evaluate to a bool. Building on our previous example, we can do some more complex matching on the individual properties of the object within the pattern. It looks something like this: \[scala\]x match { case MyThing(y,z) if y &gt; 10 =&gt; "MyThing.one is more than 10" case MyThing(y,z) if y &gt; 5 =&gt; "MyThing.one is more than 5" case \_ =&gt; "MyThing doesn't meet criteria" }\[/scala\] The `if` right after the pattern defines the guard. You can use standard boolean operators like `||` or `&&` as well, but the more complex things get the less readable things tend to be.

Hopefully I’ve given at least a little glimpse into the coolness of Pattern Matching in Scala.  The coolness of patterns is used in several different places in the language, including in the Regular Expression library, which gives a really easy way to check against regex matches, and extract elements from the regex if that’s the sort of thing you’re into.  We’ll be seeing more Pattern Matching as we start delving into more functional aspects of Scala.  Hopefully you’ll be able to appreciate the elegance and simplicity it can provide.
