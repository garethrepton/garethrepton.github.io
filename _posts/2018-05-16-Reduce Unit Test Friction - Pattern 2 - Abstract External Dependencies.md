---
layout: post
title: Patterns to reduce unit test friction - 2 - Abstract external dependencies
tags: unittesting c# patterns autofixture
---

## The Problem
Your class has a dependency on something that is outside of your control. A simple scenario for this is the DateTime object, its quite common to need to simulate a different "Now" in your tests to the one that the rest of the system uses.

e.g. 
     {% highlight CSharp %}

public class MyObject
{
    public MyObject(string name, DateTime creationDate)
    {
        this.Name = name;
        this.CreationDate = creationDate;
    }

    public string Name {get; private set;}
    public DateTime CreationDate  {get; private set;}
}

public class MyObjectFactory
{
    public MyObject Create()
    {
        return new MyObject("Testing", DateTime.UtcNow);
    }
}

     {% endhighlight %}

And the test might look like: 

     {% highlight CSharp %}
[Fact]
public void MarksMyObjectAsCreatedAtTheCurrentTime()
{
   //Arrange
   var target = new MyObjectFactory();
   //Act
   var result = target.Create();
   //Assert
   result.CreationDate.Should().Be(<What can we do here??>);
}

     {% endhighlight %}


## The Solution
Encapsulate the external dependency in a wrapper that just provides the functionality that you require to use.

So in the date case, in its simplest form, you might have:
     
     {% highlight CSharp %}

public class Clock : IClock
{
    public DateTime UtcNow { get { return DateTime.UtcNow; }}
}

public class MyObjectFactory
{
    private IClock clock;
    public MyObjectFactory(IClock clock)
    {
        this.clock = clock;
    }

    public MyObject Create()
    {
        return new MyObject("Testing", clock.UtcNow);
    }
}

     {% endhighlight %}

So now in our test we can replace the clock with a mock, and simulate the time.

     {% highlight CSharp %}
[Fact]
public void MarksMyObjectAsCreatedAtTheCurrentTime()
{
   //Arrange
   var date = DateTime.UtcNow;
   var clock = Substitute.For<IClock>();
   clock.UtcNow.Returns(date); //The important bit
   var target = new MyObjectFactory(clock);
   //Act
   var result = target.Create();
   //Assert
   result.CreationDate.Should().Be(date);
}
     {% endhighlight %}

## When its applicable
This is most applicable when dealing with non-trivial things that you can't mock, so pretty much anything talking to something external, that hasn't got a mockable interface and therefore cannot simply be replaced in the test. 

Granted DateTime might be a contrived example as you might not really care about the creation date in this case, but its actually quite a good representation of something thats not mockable or controllable , but you might want to control in an isolated way during a test.








