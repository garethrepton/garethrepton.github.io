---
layout: post
title: Async tests with NSubstitute and XUnit in C#
tags: unittesting c# patterns NSubstitute XUnit Async Await CSharp DotNet Testing
---

## Intro
This is just a really quick post on how to write async test methods in C# with XUnit and NSubstitute. Thankfully its quite simple these days, and is very useful to know as async calls become more and more commonplace.

## So how do you do it?
Say you have interface IA:

{% highlight CSharp %}

    public interface IA
    {
        Task<int> DoSomething();
    }

{% endhighlight %}

Which is called by:

{% highlight CSharp %}

    public class IAmUnderTest
    {
        public async Task<int> GetInt(IA a)
        {
            return await a.DoSomething();
        }
    }

{% endhighlight %}

## Mocking
Because Async methods just return tasks, all you need to do to mock `DoSomething()` with NSubstitute is use `Task.FromResult(<YourNumberHere>)`. Then the calling code gets a task back that it can still `await` and get back the integer result. 

So mocking it with NSubstitute looks like this:

{% highlight CSharp %}

    public class ATest
    {
        [Fact]
        public void DoesSomething()
        {
            var dependency = Substitute.For<IA>();
            dependency.DoSomething().Returns(Task.FromResult(1));

            var target = new IAmUnderTest();
            var id = target.GetInt(dependency).Result;

            id.Should().Be(1);
        }
    }

{% endhighlight %}

So its not very different to usual, you just need that Task.FromResult around the returns method.

## Async Test Methods
We can go one step further than this, and make the actual test async, because [XUnit 1.9 + has support for this](https://bradwilson.typepad.com/blog/2012/01/xunit19.html). This allows us to get rid of the .Result (shudder) from the tests and the method looks more like any other async method you might write:

{% highlight CSharp %}

    public class ATest
    {
        [Fact]
        public async Task DoesSomethingAsync()
        {
            var dependency = Substitute.For<IA>();
            dependency.DoSomething().Returns(Task.FromResult(1));

            var target = new IAmUnderTest();
            var id = await target.GetInt(dependency);

            id.Should().Be(1);
        }
    }

{% endhighlight %}

Both of these tests pass, and I think I favour the latter as its a bit more natural when testing async methods, and quite frankly seeing .Result is enough to give anyone nightmares whos come across the myriad of deadlocks it can cause.