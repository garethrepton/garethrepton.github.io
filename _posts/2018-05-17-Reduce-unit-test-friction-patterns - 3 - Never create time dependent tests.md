---
layout: post
title: Reduce unit test friction patterns - 3 - Never create time dependent tests
tags: unittesting c# patterns autofixture
---

## The Problem
There are loads of cases where this applies, but a simple one is: You have a target class that spawns a thread which updates a field with its result and you want to test that the result is set correctly. The really naive solution is to simply start the thread in your tests, do a Thread.Sleep() in there for the amount of time it takes for the result to be updated and check the result is there. This is bad in a couple of ways. 

1. It makes the test fragile.
2. By setting a time wait long enough to not be fragile, you are likely slowing down your unit tests.

Basically anything like this in a unit test is going to reduce confidence in your entire test suite.

## The Solution
There are two basic solutions, the first, and better if its possible, is to extract the logic that controls the thread into a mockable class, and make the tests run synchronously by replacing the behaviour. This completely removes the problem for the unit tests.  I won't give an example here, as it varies so much based on situation.

The second, less favourable option is to poll at short intervals to check the value is there, and set a timeout so if it doesn't ever get set, the test does still fail.

Something like ():

     {% highlight CSharp %}
[Fact]
public void MarksMyObjectAsCreatedAtTheCurrentTime()
{
   //Arrange
   var target = new MyObjectFactory();
   //Act
   target.StartWork();
   SpinWait.SpinUntil(x => target.Property != null);
   var result = target.Property;
   //Assert
    result.Should().BeTrue("Property should have been set to true");
}

     {% endhighlight %}

While this still has a side effect that it can be slow to fail, if the property isn't being set for some reason, it is still infinitely better than Thread.Sleep() as it removes the uncertainty completely.

## When to use
Whenever you feel compelled to write Thread.Sleep() in a test, use one of these.

