---
url: "/2010/06/17/announcing-the-duhking-library-for-net-3-5/"
title: "Announcing the Duhking Library for .Net 3.5"
date: 2010-06-17T21:46:05.000Z
slug: "announcing-the-duhking-library-for-net-3-5"
tags: 
  - "net"
---

If it walks like a giraffe and talks like a duck then what is it?  Maybe a duhk?  Who knows, but it certainly is not a duck.  So if that is the case, then you can probably guess what the Duhking library is all about…or maybe you can’t.  In terms of programming, [Duck Typing](http://en.wikipedia.org/wiki/Duck_Typing) refers to the ability of some languages to allow you to treat an object of one type as an object of a different type, provided the methods/properties needed exist on both objects.  Statically typed languages are usually not very good at this sort of loosey-goosey type inference, which is why this behavior is typically restricted to languages with less stringent rules on typing.

The [Duhking library](http://duhking.codeplex.com/) is an attempt to provide a very limited view of Duck Typing to .Net 3.5 applications.  It allows you to graft an interface type onto an object that has matching properties and/or methods but does not actually implement that type.  Why would you do that?  Well, there were a couple of use-cases that drove the development of this library.  One is a case where you want to wrap an API that you have no control over in an interface so that you can test your consuming code.  This is common for things like HttpContext or SmtpClient where you want to utilize the functionality of those libraries in your code which you work so hard to make testable.  A standard approach to doing this is to create an interface which defines the methods you need, and then create a “wrapper” class that implements the interface, but then calls through to the real, un-testable class to do the work.  So my thought was this:  “Since we’re just calling through to matching method signatures in a class that already exists, why not abstract the whole thing so I don’t need all of these crazy _Wrapper_ classes everywhere?”.

The other use-case came up when dealing with anonymous types.  We all know that you can create basic data objects as an anonymous type, and then use them within the scope they are created.  But what happens if you want to pass an anonymous type to another method?  Well, you have two choices.  You can either move the data in your anonymous type to another class/struct and pass _that_, or you can resort to some reflection trickery to get the values out of a plain old object.  It seemed like you should be able to create a new anonymous object as a particular interface type, and then pass it around as that interface type.

So the Duhking library allows you to do both fairly easily by use of some simple extension methods on object, and the magic of the Castle project DynamicProxy2 library.  Now that I’ve told you the secret, surely you can see how things work.  The library simply creates a proxy of the specified interface, and then intercepts calls to the interface methods, and in-turn calls the matching methods on the object we are “Duhking”.  There is some checking going on to ensure that your object is compatible with the interface you are wanting to Duhk, which involves checking method signatures (this is surprisingly complicated, considering it is the basis for compiler based interface implementation verification….but then I may be doing it the hard way), but beyond that its just passing calls off to the proxy.  But enough idle chit-chat, lets see a sample.

Okay, so lets take the first use-case where we are wanting a wrapper class for a sealed framework class so we can test our consumers.  Let’s go with the SmtpClient as an example because it is fairly common to want to send emails for various reasons.

First off we need out wrapper interface:  

```
public interface ISmtpClient
{
    string Host { get; set; }    int Port { get; set; }        void Send(string from, string recipients, string subject, string body);
}

```

You could add in additional properties or methods, but this is enough to get you going.  So now you can use this interface in place of the standard SmtpClient in the framework, and write your tests against it without major pain.  So the next step is to Duhk the real SmtpClient so it implements your interface when your ready to do the “real” work.

```
// Some code here getting ready to call your class that needs the client
var myClass = new ClassNeedingSmtpClient(new SmtpClient().AsType<ISmtpClient>());
// and now you do something with it

```

Pretty cool huh?  You can also check to see if the given concrete type can be Duhked by using the CanBe extension method

```
// Some code here getting ready to call your class that needs the client
var realClient = new SmtpClient();
if(realClient.CanBe<ISmtpClient>())
    var myClass = new ClassNeedingSmtpClient(realClient.AsType<ISmtpClient>());
// and do something else if it doesn't work

```

So now lets look at the other usage scenario, wrapping anonymous types in an iterface so you can pass them around.  The first thing we need is an interface to hold the data

```
public interface INamedSomething
{
    string Name { get; }
    int Id { get; }
    string SomethingElse { get; }
}

```

Note here that we are only specifying getters. That is because the properties of anonymous types are read-only, and right now the duhking code doesn't differentiate anonymous types from other types (more on that later). Ok, so with this we can now create an anonymous type and return it as an INamedSomething

```
public INamedSomething MethodThatReturnsSomething()
{
    // Some work goes here
    return new { Name = "Sam", Id = 1234, SomethingElse = "Hah!", SomethingElseNotInTheInterface = "Foo" }.AsType<INamedSomething>();
}

```

And this works fine. Notice I threw an extra property in there to show you that when we are checking for matching signatures we're only checking the methods/properties in the interface we're trying to Duhk. You can have as many additional properties or methods as you want in your concrete type, doesn't matter.

Now, as for that whole read-only thing.  Right now the code that checks compatability between your class and the target interface is ensuring that each method in the target interface has a matching method in the class.  This includes the compiler-generated methods for getting and setting properties.  That means that if your using an anonymous type as your class, you will never be able to Duhk it to an interface that has setters on it’s properties.  While _technically_ correct there is something about this behavior that bugs me…it just doesn’t seem flexible enough.  So most likely what I am going to do is add some special handling for anonymous types that will allow the target interface to have both getters an setters.  This will in effect provide a way to stub out an interface implementation, and use the anonymous type to set the initial values of the interface.  This does change a bit the purpose of what I’m trying to do with this library, and gives it the added ability to stub out interfaces, so I’ve held off on doing this.  I think, though, that adding this functionality will actually increase the utility of the library, so it’s probably worth doing.

Right, so now that you have all of the grueling details, [go get it](http://duhking.codeplex.com/), and let me know what you think.
