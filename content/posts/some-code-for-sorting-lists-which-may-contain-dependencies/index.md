---
url: "/2009/09/18/some-code-for-sorting-lists-which-may-contain-dependencies/"
title: "Some code for sorting lists which may contain dependencies"
date: 2009-09-18T20:08:47.000Z
slug: "some-code-for-sorting-lists-which-may-contain-dependencies"
tags: 
  - "net"
  - "tips"
---

I ran into this odd problem recently working with some Linq2SQL based persistence code.  There is some code someone put together to commit a list of changed entities to the database as part of a single transaction, which simply iterates through the list and performs the appropriate action.  The problem I was having was that I had an object referenced by another object that needed to be persisted first, otherwise there was a foreign key violation.  To add to the strangeness there seemed to be some magic going on (most likely utilizing the INotifyPropertyChanged goodness), so that even if I tried to persist just my dependent object first, both were still showing up in the list, and always in exactly the wrong order.  Now, I’m okay with magic.  Magic makes a lot of things a lot easier.  The problem arises whenever the magic is incomplete, and doesn’t follow through to take care of all of the operation.  Its like someone comming up to you and saying “Pick A Card”, at which point you do, and put the card back, and they say “I know what your card was” and walking away.  Not real convincing.  This is what was going on here.  There was the smarts to know that changes were being made to more than one entity, and there were even attributes to define what properties contained dependent objects, but no smarts to actually deal with a case when you would want to save more than one object in an object graph at a time.

So it occued to me I should be able to do some linqy magic and create some sort of iterator that would return dependent objects in the appropriate order, so the lest dependent of the objects get move to the beginning of the list.  My first step, since I wasn’t really sure how to do this, was to write a test.  And I made it more or less mirror the issue I was facing, a list of two items, one of which is a dependency of the other.  I don’t know if there is a lot of value in posting all of the test cases here, but the end result was rather nice.  Sure it took several iterations, and there was plenty of infinite looping and stack overflows (which does some fun things to studio when your running your tests with TestDriven.Net), but I think this is a reasonable solution to the problem:

```
public static IEnumerable<T> EnsureDependenciesFirst<T>(this IEnumerable<T> items, Func<T ,IEnumerable> selector)
{
    if(items.Count() < 2)
        return;
    var firstPass = items.SkipWhile(t => items.Intersect(selector(t)).Count() > 0);
    var remainingItems = items.Except(firstPass);
    if(items.Count() == remainingItems.Count())
        return remainingItems;
    return firstPass.Concat(remainingItems.EnsureDependenciesFirst(selector));
}

```

Ok, so what do we have here?  Well to start out I’m checking the item list to see if there are at least two items in it, if not I just return the list.  This provides a means to avoid an infinate loop due to the recursive call, and provides a shortcut for a scenario with only one item.  Next off I use the SkipWhile() method, combined with the user-supplied selector function to iterate through each item, retrieve it’s list of dependencies (which is what the selector function does), and checks to see if the current list contains any of the dependencies for the object.  The results of this first pass are the objects which have no dependencies at all, so therefore they need to be first in the list.  The next logical step is to run the operation again for a list that **does not** contains the items filtered out by the first pass.  This is done via a recursive call back to the EnsureDependenciesFirst extension.  You will notice we’re checking the count of the remaining items against the current list, and returning the list if they are the same.  This is another safety precaution for dealing with infinite loops.  If we have a circular dependency, this bit will just return the items that are interdependent.

You will note that this is a generic function that has really noting at all to do with the entities that I am dealing with.  This was largely due to the fact that this was built TDD, so I just used a simple class which had a property that could take another instance of itself.  To use this to overcome my entity committing problem, I would have to write a not too small function to retrieve the list of dependent objects from the entity (since there would need to be some reflection magic to look at attributes on the properties to determine which properties contain dependencies), but it pretty much will drop in to the foreach statement that is currently being used to persist the entities.

Incidently, I learned from my dev team what the “official” way of dealing with this is a “ReorderChanges” method, which takes two entities, in the order in which they should be persisted.  I think I like my solution better, mostly because it should mean I don’t have to worry about it again.
