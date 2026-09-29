---
url: "/2015/06/08/foldthe-greatest-thing-that-ever-happened-to-your-data-structure/"
title: "fold - The greatest thing that ever happened to your data structure"
date: 2015-06-08T17:16:22.000Z
slug: "foldthe-greatest-thing-that-ever-happened-to-your-data-structure"
tags: 
  - "net"
  - "f"
  - "functional-programming"
  - "tips"
---

Let's say that you’ve been working hard on this really awesome data structure. Its fast, its space efficient, its immutable, its everything anyone could dream of in a data structure. But you only have time to implement one function for processing the data in your new miracle structure, so what would it be?

Ok, not a terribly realistic scenario, but bare with me here, there is a point to this. The answer to this question, of course, is that you would implement `fold`. Why you might ask? Because if you have a fold implementation then it is possible to implement just about any other function you want in terms of fold. Don’t believe me? Well, I’ll show you, and in showing you I’ll also demonstrate how finding the right abstraction in a functional language can reduce the size and complexity of your codebase in amazing ways.

Now, to get started, let's take a look at what exactly the fold function is:

\[fsharp\] val fold:folder('State -> 'T -> 'State) -> state:'State -> list:'T list -> 'State \[/fsharp\]

In simple terms it iterates over the items in the structure, and applies a function to each element which in some way processes the element and returns some kind of accumulator. Ok, maybe that didn’t come through quite as simply as I would have hoped. So how about start with a pretty straight-forward example: `sum`.

\[fsharp\] let sum (list:int list)= List.fold (fun s i -> s + i) 0 list \[/fsharp\]

Here we are folding over a list of integers, but in theory the data structure could be just about anything. Each item in the list gets added to the previous total. The first item is added with the value passed in to the `fold`, so for items `[1;2;3]` we start by adding `1` to `0`, then `2` to `1`, then `3` to `3`, the result is `6`. We could even get kinda crazy with the point-free style and use the fact that the `+` operator is a function which takes two arguments, and returns a third…which happens to exactly match our folding function.

\[fsharp\] let sum (list:int list) = List.fold (+) 0 list \[/fsharp\]

So that’s pretty cool right? Now it seem like you could also very easily create a Max function for your structure by using the built in `max` operator, or a Min function using the `min` operator.

\[fsharp\] let max (list:int list) = List.fold (max) (Int32.MinValue) list let min (list:int list) = List.fold (min) (Int32.MaxValue) list \[/fsharp\]

But I did say that you could create any other processing function right? So how about something a little trickier, like Map? It may not be quite as obvious, but the implementation is actually equally simplistic. First let's take a quick look at the signature of the `map` function to refresh our memories:

\[fsharp\] val map: mapping ('T –> 'U) –> list:'T list –> 'U list \[/fsharp\]

So how do we implement that in terms of fold? Again, we’ll use List because its simple enough to see what goes on internally:

\[fsharp\] let map (mapping:'a -> 'b) (list:'a list) = List.fold (fun l i –> mapping i::l) \[\] list \[/fsharp\]

Pretty cool right? Use the Cons operator (`::`) and a mapping function with an initial value of an empty list. So that’s pretty fun, how about another classic like `filter`? Also, pretty similar

\[fsharp\] let filter (pred:'a -> bool) (list:'a list) = List.fold (fun l i –> if pred I then i::l else l) \[\] list \[/fsharp\]

Now we’re on a roll, so how about the `choose` function (like `map`, only returns an `Option` and any None value gets left out)? No problem.

\[fsharp\] let choose (chooser:'a –> 'b option) (list:'a list) = List.fold (fun l i –> match chooser i with | Some i –> i::l | \_ –> l) \[\] list \[/fsharp\]

Ok, so now how about `toMap`?

\[fsharp\] let toMap (list:'a\*'b list) = List.fold (fun m (k,v) –> Map.add k v) Map.empty list \[/fsharp\]

And `collect` (collapsing a list of lists into a single list)?

\[fsharp\] list collect (list:'a list list) = List.fold (fun l li –> List.fold (fun l' i' –> i'::l') l li) \[\] list \[/fsharp\]

In this case we’re nesting a fold inside a fold, but it still works. And now, just for fun, list `exists`, `tryFind`, `findIndex`, etc

\[fsharp\] let exists (pred:'a -> bool) (list:'a list) = List.fold (fun b i -> pred i || b) false list let tryFind (pred:'a -> bool) (list:'a list) = List.fold (fun o i -> if pred i then Some i else o) None list let findIndex (pred:'a -> bool) (list:'a list) = List.fold (fun (idx,o) i -> if pred i then (idx + i,Some idx) else (idx + 1,o)) (-1,None) list |> snd |> Option.get let forall (pred:'a -> bool) (list:'a list) = List.fold (fun b i -> pred i && b) true list let iter (f:'a -> unit) (list:'a list) = List.fold (fun \_ i -> f i) () list let length (list:'a list) = List.fold (fun c \_ -> c + 1) 0 list let partition (pred:'a -> bool) (list:'a list) = List.fold (fun (t,f) i -> if pred i then i::t,f else t,i::f) (\[\],\[\]) list \[/fsharp\]

Its worth pointing out that some of these aren’t the most efficient implementations. For example, `exists`, `tryFind` and `findIndex` ideally would have some short-circuit behavior so that when the item is found the list isn’t traversed any more. And then there are things like `rev`, `sort`, etc which could be built in terms of fold, I guess, but the simpler and more efficient implementations would be done using simpler recursive processing. I can’t help but find the simplicity of the fold abstraction very appealing, it makes me ever so slightly giddy (strange, I know).
