---
layout: post
title: Patterns to reduce unit test friction - 1 - Avoid explicit construction
tags: unittesting c# patterns autofixture
---

## The Problem
Anyone who's written unit tests that have had to change will probably be familiar with this, objects that tend to interface to other classes tend to take in more parameters during the development cycle every now and then, if you've just newed up using the standard approach e.g.

    {% highlight CSharp %}
        var target = new MyTestObject(new MyDependency(), "Something");
    {% endhighlight %}

You will have to modify every single test method that news up that class to cope with its new interface, this can make the keeping the tests working and up-to-date feel like more of a chore than it has to be.

## The Solution
Abstract away the construction logic, this is standard practice for a normal application, I think for any non-trivial objects being tested it should be for those too. 

There are lots of ways you can do this, but I really like the library [AutoFixture](https://github.com/AutoFixture/AutoFixture) for this, as you can tell it what to use when a certain dependency is required for your class, and it will automatically use that for anything that requires it.  It will also inject in pseudorandom values for simple types for you, without any setup whatsoever.

So:
    {% highlight CSharp %}
        var target = new MyTestObject(new MyDependency(), "Something");
    {% endhighlight %}
becomes:
    {% highlight CSharp %}
        var target = fixture.Create<MyTestObject>();
    {% endhighlight %}

## When its applicable
Use when the class under test is non trivial and has a strong possibility of its construction logic changing frequently, or you are just starting out creating the class under test.






