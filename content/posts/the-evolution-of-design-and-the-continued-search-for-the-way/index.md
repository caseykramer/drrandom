---
url: "/2007/10/02/the-evolution-of-design-and-the-continued-search-for-the-way/"
title: "The evolution of design, and the continued search for \"The Way\""
date: 2007-10-02T20:53:00.000Z
slug: "the-evolution-of-design-and-the-continued-search-for-the-way"
tags: 
  - "ioc"
---

I'm currently finding myself in the midst of an evolutionary change.  And I'm not talking about my super-human mutant powers, I'm talking about the way I'm thinking about solving a specific set of problems.

Let's start with a sample....Lets take something like processing credit-cards as a benign and IP free place to start.  As a subject that I really have no practical experience with, it seems like an appropriate choice.  I'm going to assume that there are different rules for doing checksum validation on credit card numbers, depending on what the card is (Mastercard/Visa/Discover/etc). Now, here is evolutionary step 1: Use a basic case statement to process the various cards.  Here is what something like that would look like:

```
switch(CardType)
{
    case CardType.MasterCard:
        CardValidators.MasterCardValidator(card.CardNumber);
        break;
    case CardType.Visa:
        CardValidators.VisaValidator(card.CardNumber);
        break;
    case CardType.Discover:
        CardValidators.DiscoverValidator(card.CardNumber);
        break;
}
```

This looks pretty straight-forward, and as it stands it isn't too bad from a maintainability stand point.  But what happens when there are many different types of cards?  An then what happens when you find a large amount of duplication between the validation functions?

Well, any student of GoF should be able to tell you that a Chain Of Responsibility pattern looks like a perfect fit for this sort of scenario.  So, evolutionary step 2: Create separate classes to handle the different types of validation, and configure them in a Chain of Responsibility pattern where each instance decides for itself whether it can process the input.

Here is a quick and dirty look at what something like that would look like:  
<!-- missing image: /image.axd?picture=transfer/ClassDiagram1.png -->

The two most interesting things here are the GetValidator() method in the AbstractCardValidator, and the individual CanValidate() methods in the concrete implementations.  What this does is it allows each class to decide for itself how it is going to determine whether or not it can be used as a validator for a specific card (thats the CanValidate() part), and also provides a single point which the consumer of the API can use to get _the_ validator for the card instance they have.  You would probably want to build and Abstract Factory around this, which would instantiate all of the ICardValidator classes, and then run the GetValidator() method to get the correct one.

Now we are at a point where things are looking pretty good; we've got the ability to do some fairly complex logic to make the decision about which validator to use, and we have a way to simply ask for one, and the correct one appears.  Pretty cool.

This is actually the place where I have found myself in the not to distant past.  I have previously been perfectly content with this arrangement, and been fairly happy with the separation of concerns among the classes...I mean, after all, who better to decide whether or not a specific class should be used to validate a card than the class itself.  So what is the issue?  Well, recently I have become aware of two problems with this arrangement: Tight Coupling, and a violation of the Single Responsibility Principle.  Let's start with the first:

**Tight Coupling  
**The credit card example may be a bit contrived when it comes to this issue, but bear with me.  Overall, the issue is that specific instances of ICardValidtor objects are being created and handed around.  The use of an interface and an Abstract Factory pattern would actually help the situation out some, but effectively all it does is move the coupling from the consuming class to the Factory (okay, it also consolidates coupling to a single class, which makes maintenance a lot easier).  As I said, contained, but still there.  It would be nice if the factory didn't need any knowledge of what concrete instances of ICardValidator were out there.  Before we tackle that, though, lets also look at the second issue:

**Violation of the "Single Responsibility Principle"  
**The SRP states that a class should have one, and only one, thing it is responsible for.  Sounds pretty easy doesn't it?  The problem is that this can be difficult to obtain without a fair amount of discipline.  The violation of SRP which I'm seeing is that the ICardValidator is responsible for both validating a credit card _and_ determining which validator is appropriate.  But wait!  Didn't I just say that moving this check into the ICardValidator instance was a "Good Thing"?  Well, lets go as far as saying it is _better_ than the previous method, but still not perfect.  Applying the SRP would move the task of selecting a validator from the ICardValidator instance, and put it on it's own somewhere.  So, thusly we come to our:

**Inversion Of Control Container**.  
That's right, we are now going to get crazy and move the responsibility of creating these instances to another component all together.  The nice thing about this is that it allows us to move all of the knowledge about dependencies off somewhere else.  How does this apply to this example?  Well, lets assume we have an object of type _Card_ which requires as a dependency an instance of an _ICardValidator._  We'll also assume that _Card_ is subclassed based on the type of credit card.  It now becomes trivial to configure our IoC container to supply a _specific implementation_ (read sub-type) of _ICardValidator_ for each implementation (again, read sub-type) of _Card_.  Now, when you want a _Card_ instance, you ask the IoC container for one, and depending on what type of card it is, you will get the appropriate _ICardValidator_ as well.

What's the catch?  Well there is some additional complexity which will show up somewhere in the application due to the IoC, but typically IoC configuration can be delegated down to the configuration file level, so even then the ugliness is pushed away to it's own dark corner.

But wait!  Why should we have different instances of _Card_?  What if the Card class is just a container for the card data?  Well, our Ioc still gives us some advantages.  If we look back at our first example with the switch statement, we've got a nice _CardType_ enum, which could be a property of our _Card_ class.  Using an IoC container like the one provided by the Castle project, you have the ability to configure a key string for your instances.  This would make it trivial to map the enum choices to specific keys within the container, which the _Card_ class would use to get an ICardValidator instance.  This would also make it possible to make the validators slightly more advanced by adding something like a Decorator pattern, in which specific aspects of the validation could be factored into separate classes, and then "stacked" to produce the final validation logic (This is the same concept used by the Stream classes in .Net and Java.  You can modify the behavior of a stream by passing it to the constructor of a stream with different behavior).

It is definatly worth mentioning that there is a sudden appearance of tight coupling to the IoC container itself from our consuming classes.  You probably want to try to abstract away the fact that the IoC container exists from the majority of the application.  Factory classes go a fair ways in making this happen, but another good idea is to introduce a single service to do type resolution.  The Factory classes can then ask this service for the object they want, and they never need to know the IoC container is there.  This approach also gives you the ability to create some objects using IoC and others in another (more traditional) way.

So is this it?  Have I finally found the answer I've been looking for?  It's hard to say right now.  For the time being this is a decent way to handle things, provided the complexity of the underlying system, and the need for loose-coupling are both high enough to justify the additional complexity of the IoC.  But who knows, in another couple months I may find something new, or even something old, which seems better, cleaner, simpler.  That, after all, is my final goal....And I need to remind myself of that regularly, lest I become complacent.
