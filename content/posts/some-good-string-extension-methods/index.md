---
url: "/2009/02/22/some-good-string-extension-methods/"
title: "Some (good?) string extension methods"
date: 2009-02-22T17:19:09.000Z
slug: "some-good-string-extension-methods"
tags: 
  - "uncategorized"
---

So, to take a slightly different turn from my usual meta discussions of process, theory, and architecture, there have been several people who have offered up some examples of extension methods that they have found useful, now that .Net 3.5 is roaring along nicely.  There are some collections of such utilities, like the [Umbrella project](http://www.codeplex.com/umbrella), and some folks like [Bill Wagner](http://srtsolutions.com/blogs/billwagner/default.aspx) who have [written books](http://www.amazon.com/More-Effective-Specific-Software-Development/dp/0321485890) on the subject (okay, there are other things in there too), so I thought I might as well throw my hat into the ring as well.  Specifically, there was this [tweet](http://twitter.com/elijahmanor/status/1219521290) from [@elijahmanor](http://twitter.com/elijahmanor) a few days ago.  It points to a positing which includes some extensions on string to convert from strings to value types (int, long, short, etc).  I pointed out that in our current project we have distilled this down to a single extension method: To().

He suggested I blog about it, so here it is:

So we actually have two classes of conversions, one converts from a string to a value type, and they other converts from a string to a Nullable value type.  In the case of our project, the nullable version came first, and so it became very easy to create the version that returned a non-nullable value.  Here are the methods of interest: \[csharp\] public static T To(this string input) where T : struct { return input.To(default(T)); }

public static T To(this string input, T defaultValue) where T: struct { return input.ToNullable() ?? defaultValue; }

public static T? ToNullable(this string input) where T : struct { if (string.IsNullOrEmpty(input)) return null; var tryParse = GetTryParse(); return (T?)tryParse(input); } \[/csharp\] Okay, so this is pretty straight forward.  The non-nullable version calls through to the nullable version, and if it is null, it returns default(T).  But, as I'm sure you are an astute reader, you will see that there is some magic going on; namely the GetTryParse() method.  This little guy goes off and looks for a TryParse method on whatever T happens to be, and then returns a Func<> delegate that will run the string input through the try parse and return either a null (in case the TryParse fails), or a boxed version of the result.  So, lets see what it looks like before we discuss pros and cons \[csharp\] private static Func<string,object> GetTryParse() { var tryParseEx = GetTryParseExpression(); return (s) =&gt; tryParseEx.Compile()(s,default(T)); }

private static Expression&lt;Func&lt;string,object&gt;&gt; GetTryParseExpression() { if (\_tryParseCache.ContainsKey(typeof(T))) return \_tryParseCache\[typeof(T)\] as Expression&lt;Func&lt;string, T, object&gt;&gt;;

MethodInfo tryParse = typeof(T).GetMethod("TryParse", new Type\[\] { typeof(string), typeof(T).MakeByRefType() }); Ensure.IsNotNull(tryParse, string.Format("Cannot convert from type string to type {0} because {0} does not have a TryParse method", typeof(T).FullName));

var stringArg = Expression.Parameter(typeof(string), "input"); var tempArg = Expression.Parameter(typeof(T), "tmp");

var tryParseEx = Expression.Lambda&lt;Func&lt;string,object&gt;&gt;( Expression.Condition( Expression.Call(tryParse, stringArg, tempArg) , Expression.Convert(tempArg, typeof(object)) , Expression.Constant(null)) , stringArg, tempArg); \_tryParseCache.Add(typeof(T), tryParseEx); return tryParseEx; } \[/csharp\] So here we have some code looking for a `TryParse` method, and building an expression (using Linq Expression Trees) to execute it.  Now, I'll be honest, this is not the code I'm using in the project where this originally came from...mostly because I didn't think of it then.  In that case I'm actually doing a big case statement, checking the type of T and running the appropriate method.  This is much shorter, but potentially much slower at runtime.  So that is where the \_tryParseCache comes in.  This is a simple static dictionary which contains the expressions created for each of the types, which means you only get the runtime performance hit once when you first ask to parse a specific type.  The declaration for this object looks like this: \[csharp\] private static Dictionary \_tryParseCache = new Dictionary(); \[/csharp\] There you have it, my first (and possibly last??) contribution into the world of extension methods. Please commence criticisms
