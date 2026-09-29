---
url: "/2012/07/24/purely-functional-data-structures-part-2/"
title: "Purely Functional Data Structures–Part 2"
date: 2012-07-24T18:15:17.000Z
slug: "purely-functional-data-structures-part-2"
tags: 
  - "net"
  - "c"
  - "f"
  - "functional-programming"
---

So here we are at part 2 in the series of posts looking at Functional Data Structures from the book of the same name by Chris Okasaki. [Last time](/2012/07/23/purely-functional-data-structures-part-1/) we looked at what is perhaps the simplest of the functional data structures, the List (also useful as a LIFO stack).  Up next we’ll continue in the order that Chris Okasaki used in his book, and take a look at implementing a Set using a Binary Tree.

Diving right in, here is implementation for a Set using a binary tree in F#: \[fsharp\] module Set

type Tree<'a when 'a:comparison> = | Empty | Tree of Tree<'a>\*'a\*Tree<'a>

let rec isMember value tree = match (value,tree) with | (\_,Empty) -> false | (x,Tree(a,y,b)) -> if value < y then isMember x a elif value > y then isMember x b else true

let rec insert value tree = match (value,tree) with | (\_,Empty) -> Tree(Empty,value,Empty) | (v,Tree(a,y,b)) as s -> if v < y then Tree(insert v a,y,b) elif v > y then Tree(a,y,insert v b) else snd s \[/fsharp\] This is pretty simple, like the List we're working with a Discriminated Union, this time with an Empty, and then a Tree that is implemented using a 3-tuple (threeple?) with a Tree, an element, and a Tree. There is a constraint on the elements that ensures they are comparable, since this is going to be an ordered tree.

We only have two functions here, one isMember, which says whether or not the element exists in the set, and the other insert, which adds a new element. If you look at the isMember function, its not too difficult, a recursive search of the tree attempting to find the element. Since this is a sorted tree, each iteration will compare the element being searched for with the element in the current node of the tree. If its less than the current node we follow the right-hand side of the tree, otherwise we follow the left-hand side of the tree. If we find an empty tree, the element doesn't exist. Update is a little more difficult...it's recursive like isMember, but it is also copying some of the paths. The bits that are copied are the bits that are **not** being traversed, so in reality the majority of the tree returned from the update function is actually shared with the source tree, its root is just new. Take a hard look at that for a moment, and see if the pain begins to subside...then we'll look at the C# version. \[csharp\]public static class Tree { public static EmptyTree<T> Empty<T>() where T: IComparable { return new EmptyTree<T>(); } }

public class EmptyTree<T> : Tree<T> where T: IComparable { public override bool IsEmpty { get { return true; }} }

public class Tree<T> where T: IComparable { public Tree<T> LeftSubtree { get; internal set; } public Tree<T> RightSubtree { get; internal set; } public T Element { get; internal set; } public virtual bool IsEmpty { get { return false; } } }

public static class Set { public static bool IsMember<T>(T element, Tree<T> tree) where T: IComparable { if (tree.IsEmpty) return false; var currentElement = tree.Element; var currentTree = tree; while(!currentTree.IsEmpty) { if (element.CompareTo(currentElement) == 0) return true; if (element.CompareTo(currentElement) == 1) { currentTree = currentTree.RightSubtree; } else { currentTree = currentTree.LeftSubtree; } currentElement = currentTree.Element; } return false; }

public static Tree<T> Insert<T>(T element, Tree<T> tree) where T: IComparable { if (tree.IsEmpty) return new Tree<T> { LeftSubtree = Tree.Empty<T>(), Element = element, RightSubtree = Tree.Empty<T>() }; switch(element.CompareTo(tree.Element)) { case 0: return tree; case 1: return new Tree<T> { RightSubtree = tree.RightSubtree, Element = tree.Element, LeftSubtree = Set.Insert<T>(element,tree.LeftSubtree) }; default: return new Tree<T> { LeftSubtree = tree.LeftSubtree, Element = tree.Element, RightSubtree = Set.Insert<T>(element, tree.RightSubtree) }; } } }\[/csharp\] This is a reasonable chunk of code, so lets work it from the top down. We start off by defining the Tree data structure. We use inheritance in this case to make an Empty tree, since we don't have Discriminated Unions in C# (If I were a good person I would update that right now to return a singleton of the EmptyTree class, but alas, I'm lazy). The Static Tree class provides the convenience method for creating the empty tree, and the Tree type is our parameterized tree.

The methods in the Set class do the work of checking for an existing member in the set, and inserting a new member in the set.  I took the opportunity to convert the recursive isMember function to a looping construct in C# (which is what the F# compiler will do for you).  This is not really possible with the Insert method because it is not tail recursive.  The logic is the same in both versions, but the C# version is a bit more verbose (though having LeftSubtree and RightSubtree makes things a little clearer in my opinion).  Again, the biggest difference between the two is the amount of code (since we don’t have Discriminated Unions and Pattern Matching in C# land)

#### Summing Up Persistent Structures

Interestingly this is where the first section of Okasaki’s book ends (Its actually chapter 2, but chapter 1 is more of a foundational thing…no code).  These two implementations show the basic ideas behind what are described as “Persistent” data structures…meaning bits of the structures are re-used when creating new structures are part of an operation that would mutate the original structure in a non-functional (mutable) data structure.  In the case of a List/Stack we are referencing the old list as the “Tail” of the new list, so each time we add a new item we are simply allocating space for the new item.  In the case of the Tree/Set we create a new root tree on Add, and then reference all paths except for the new node that gets added (or, if the item already exists, we just have the new root…this is actually something Okasaki suggests the reader should solve as an additional exercise).  These concepts are fundamental to the more complex data structures that fallow, and present the basic ideas that are employed to make the structures efficient within the context of functional programming.

Up next in the book is a look at how more traditional data structures, such as heaps and queues, can be converted to a more functional setting.  Expect more goodness in the area, but I would also like to revisit some of the basics here.  The more observant readers may have noticed that the majority of the functions used on these simple types were not Tail Recursive, which means the compiler and JIT cannot optimize them, which ultimately means they are going to cause your stack to blow up if you’re dealing with large structures.  It might be worth exploring how to go about converting these to make them Tail Recursive.
