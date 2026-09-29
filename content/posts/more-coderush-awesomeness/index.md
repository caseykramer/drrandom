---
url: "/2009/12/15/more-coderush-awesomeness/"
title: "More CodeRush Awesomeness"
date: 2009-12-15T15:08:00.000Z
slug: "more-coderush-awesomeness"
tags: 
  - "c"
  - "tdd"
---

On November 27th, a beta release of the 9.3 version of the Developer Express components, including CodeRush and Refactor Pro! was made available to subscribers.  This release is pretty significant to me because it contains a major feature that I have been waiting for [for a long time](/2007/07/13/taking-the-coderush-plunge/): A Unit Test Runner.  There were some teasers released by [Mark Miller](http://community.devexpress.com/blogs/markmiller/) a while back, which only made me want to get my hands on the tool that much more.  My initial impressions are that it is very nice.  It is similar to [TestDriven.Net](http://www.testdriven.net) in that it provides context menu options to run tests at various levels of granularity (single test, file, project, and solution level) and includes a debug option.  At this point it does not contain some of the additional coolness that TestDriven gives you like NCover/Team Coverage and TypeMock integration, but it does have the advantage of being extensible.  I know it was extensible because Mr. Miller [told me it was extensible](http://community.devexpress.com/blogs/markmiller/archive/2009/11/16/the-test-runner-you-ve-been-waiting-for.aspx) (the title “The Extensible Unit Test Runner You’ve Been Waiting For” was a clue).  I did not realize how extensible, however, until after I submitted a [bug report](http://www.devexpress.com/issue=B142663) to DevExpress.  The bug I was reporting (the NUnit TestCase attributes were not recognized), it turns out, was already brought to the attention of the DX team by way of a [forum post](http://community.devexpress.com/forums/p/83453/285881.aspx#285881), and they had already planned on correcting it with the next 9.3 release, but I could have saved myself (and Vito on DevExpress team) some time by taking a peek at the source samples bundled with the 9.3 release.  Yep, you guessed it, there with a shared source license were all of the test framework implementation projects.  So this meant I could whip together my own temporary fix while I was waiting for the next release.  It seemed like something that other folks might want to know about, so I thought I would share it here.

The biggest piece of the puzzle is a new TestExecuteTask class for handling the TestCaseAttribute.  Due to my complete lack of creativity, I called mine TestCaseExecuteTask, and it looks like this:

```
using System;
using System.Collections.Generic;
using System.Text;
using DevExpress.CodeRush.Core.Testing;
using System.Reflection;
using DevExpress.CodeRush.Core;

namespace CR_NUnitTesting
{
    public class TestCaseExecuteTask : TestExecuteTask
    {
        public override TaskExecuteResult CollectTestParameters()
        {
            TaskExecuteResult result = TaskExecuteResult.SkippedTaskResult;
            Attribute testCase = GetMethodAttribute("NUnit.Framework.TestCaseAttribute");
            if (testCase == null)
                return result;
            
            foreach(Attribute testCaseItem in TestMethod.GetCustomAttributes(true))
            {
                if(testCaseItem == null)
                    continue;
                var testCaseType = testCaseItem.GetType();
                if(testCaseType == null || testCaseType.FullName != "NUnit.Framework.TestCaseAttribute")
                    continue;
                PropertyInfo prop = testCaseType.GetProperty("Arguments");
                if(prop == null)
                    continue;
                foreach(MethodInfo getter in prop.GetAccessors())
                {
                    object[] parameters = getter.Invoke(testCaseItem, Type.EmptyTypes) as object[];
                    result.AddParameters(parameters);
                }
            }
        }
    }
}

```

This could be cleaned up some, and some of the magic strings extracted to constants, but overall it is pretty simple. Basically what is going on here is that we are looking for the TestCase attribute, and extracting the arguments for any attributes we find.  It just so happens that the TestExecuteTask base class has a CollectTestParameters() method we can override which allows for this sort of Row testing.  The parameters we extract get stashed in the execution result, which causes the test runner to execute the test once for each group of parameters (the result has a list of parameters, which gets populated with an array of objects for each TestCase attribute), and will correctly display which cases failed if there is a failure.

There are a couple other small changes that need to happen to get this to work.  There is an NUnitExtension.cs  class, which is the Plug-In class for the NUnit support, and it handles wiring everything up for us.  First off we need to initialize our new TestExecuteTask, and add it to the list of tasks that run for NUnit tests.  We do that in the InitializePlugin method of the NUnitExtension class:

```
public override void InitializePlugin()
{
    base.InitializePlugin();
    nUnitProvider.AvailableTasks.Add(new NUnitIgnoreTask());
    nUnitProvider.AvailableTasks.Add(new NUnitSetupTearDownTask());
    nUnitProvider.AvailableTasks.Add(new NUnitExpectedExceptionTask());
    nUnitProvider.AvailableTasks.Add(new NUnitValuesTask());
    nUnitProvider.AvailableTasks.Add(new NUnitRowTestTask());
    nUnitProvider.AvailableTasks.Add(new NUnitTimeoutTask());
    nUnitProvider.AvailableTasks.Add(new NUnitExplicitTask());
    nUnitProvider.AvailableTasks.Add(new NUnitTestCaseTask());
}

```

Ours gets added to the end of the list, so it will be executed. The next step is to get the plug-in to realize that a method with a TestCase attribute is an executable test method. That trick happens in the handler for the CheckTestMethod event on the UnitTestProvider. All we're going to do is add another condition to an if statement like so:

```
void nUnitProvider_CheckTestMethod(object sender, CheckTestMethodEventArgs ea)
{
    IMethodElement method = ea.Method;
    if(//method.Name != null && method.Name.StartsWith("Test")
       ea.GetAttribute("NUnit.Framework", "Test", method) != null
    || ea.GetAttribute("NUnit.Framework.Extensions", "RowTest", method) != null
    || ea.GetAttribute("NUnit.Framework", "TestCase", method) != null)
    {
        ea.IsTestMethod = true;
        ea.Description = ea.GetAttributeText("NUnit.Framework", "Description", method);
        ea.Category = ea.GetAttributeText("NUnit.Framework", "Category", method);
    }
}

```

The only change to the original code was the additional GetAttribute call at the end of the if statement (the comments were there when I got there, I swear).  Now the only thing left to do is to compile it and drop it in the plug-ins directory.  Now when you are looking at a test class, you should be able to run TestCase decorated test methods without problem.  Well, almost.  There is one thing I was not able to find a clean way to implement, and that is the Result property of the TestCase attribute.  This allows you to streamline tests which are doing equals assertions by having the test method return the actual result, and you specify the expected result by using the result property.  Unfortunately I could not find a way to hook into the actual execution of the test in such a way that I could have access to the specific test properties being used, and the result of the test method execution.  But considering the DevExpress folks will be fixing this issue, I’m sure when they release it there will be support for this feature.  After all, this is simply a stop-gap solution until the next CodeRush release is available, so I’m willing to live with this slight inconvenience.

Happy Testing!
