---
url: "/2012/07/23/purely-functional-data-structures-part-1/"
title: "Purely Functional Data Structures–Part 1"
date: 2012-07-23T06:00:00.000Z
slug: "purely-functional-data-structures-part-1"
tags: 
  - "c"
  - "f"
  - "functional-programming"
---

I thought it might be fun to explore a little bit of CS as it applies to functional programming, by looking at the idea of Functional Data Structures.  This is actually an area that is still getting a lot of active research, and is pretty interesting stuff overall.  The general idea is to try and figure out ways to provide immutable data structures which can be efficiently implemented in a functional setting.  So you look at some standard data structures, like a linked list, and find a way to implement that as an immutable linked list.  One of the really cool features of Functional Data Structures is that because your dealing with them in an immutable setting, you can actually get a lot of re-use out of them….specifically for something like a list, you can add an item to the list, and return a “new” list that consists of the old list and the new item, and literally provide a structure that points to the old list instead of copying items.  Even if you have other parts of the code referencing older versions of the list without the new item, you don’t have to worry since none of them can mutate the list.

The biggest body of research on this topic was published by Chris Okasaki in 1998, and is still the definitive reference on the subject today.  Just for fun I’m going to look at some of the structures discussed in the original book and see what the implementations would look like in F# and C#.  The original text provided samples in Standard ML, with an eppendix containing Haskell versions.  I won’t go into too much depth on the theory behind the structures, but I will try to point out the interesting bits.

Without further ado, lets get rolling with our first data structure, which is also Okasaki’s first: Lists

Specifically, we’re going to implement a singly-linked list, which can be used rather effectively as a LIFO stack.  To start off lets look at the F# version of the list, which is closest to what Okasaki listed in his book.  The basic list type looks like this:

\[fsharp\] type List<'a> = | Empty | Cons of 'a \* List<'a> \[/fsharp\]

This is a simple Discriminated Union, with two options, Empty, and something I’ve called Cons in honor of the Lisp folks. The Cons option is basically a tuple containing an element of type type ‘a, and a List of ‘a.  This by itself is reasonably uninteresting, so lets actually do something with this. \[fsharp\] let isEmpty = function | Empty -> true | \_ -> false

let cons head tail= Cons(head,tail)

let head = function | Empty -> failwith "Source list is empty" | Cons(head,tail) -> head

let tail = function | Empty -> failwith "Source list is empty" | Cons(head,tail) -> tail

let rec (++) leftList rightList = match leftList with | Empty -> rightList | Cons(head,tail) -> Cons(head,tail ++ rightList)

let rec update list index value = match (list,index,value) with | (Empty,\_,\_) -> failwith "Source list of empty" | (Cons(\_,tail),0,v) -> Cons(v,tail) | (Cons(\_,tail),i,v) -> update tail (i - 1) v \[/fsharp\] Here we have some basic functions, an isEmpty check, a cons method (which creates a list), the head and tail functions, along with a ++ function, which appends two lists, plus an update method which changes the value of a particular element in the list.  Notice the update and ++ functions are both recursive, and in the case of the ++ function, it is **not** tail recursive. This is probably ok in this case since the performance of the ++ function is O(n) where n = length of the left list.  Both of these functions are also interesting because the F# compiler is unable to optimize them by converting them into a loop.

If we look at the C# version of these same structures things look pretty much the same: \[csharp\]public static class List { public static List<T> Empty<T>() { return new EmptyList<T>(); }

public static List<T> Cons<T>(T head, List<T> tail) { return new List<T> { Head = head, Tail = tail }; } } public class EmptyList<T> : List<T> { public bool IsEmpty { get { return true; } } }

public class List<T> : List { public T Head {get; set; } public List<T> Tail {get; set; }

public bool IsEmpty { get { return false; } }

public List<T> Update(int index, T value) { if(this.IsEmpty) throw new InvalidOperationException("You can't update an empty list"); if(index == 0) return List.Cons<T>(value,this.Tail); return this.Tail.Update(index - 1, value);

}

public static List<T> operator +(List<T> leftList, List<T> rightList) { if(leftList.IsEmpty) return rightList;

return List.Cons<T>(leftList.Head, leftList.Tail + rightList); } } \[/csharp\] Other than being almost twice as long, there are not many differences between the C# version of this structure and the F# version In this version I’ve opted to make the empty list a subclass of the List that has the IsEmpty property return true all the time.  There is also a static Empty<T>() method which returns an empty list.  A reasonable improvement could be to make this a singleton, so that empty lists would also share reference equality. Since the ++ operator in C# is not overloadable (and is a unary operator to boot) I’ve used an overload of the + operator for concatenating two lists.  The implementations are the same as the F# versions, though honestly recursion is a little strange in C#.  We still have the same performance characteristics, where appending an element is an O(n) operation, We also have the same issues with recursion, namely a stackoverflow if we have a large enough list.  Though, honestly with the performance of the update operation overall, you should probably find a new structure before you get to the point where your going to overflow your stack.

One very nice use for this particular structure is the LIFO stack.  Rather than the typical “push” and “pop” operations, we have the “cons” and “head”/”tail” operations (in the case of pop, you have “head” which gives you the elements, and “tail” which gives you the rest of the list).  This works well because pushing and popping are O(1).  This structure is not all that different than the built-in List type in F#, without the benefit of the additional functions (filter, map, tryFind, etc).  Thought it would be reasonably trivial to implement these in a recursive fashion.

 

That’s it for this segment…up next we’re going to look at using an immutable binary tree to implement a Set….good stuff for sure.
