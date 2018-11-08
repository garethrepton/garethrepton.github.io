---
layout: post
title: Reduce unit test friction patterns - 5 - Autofixture as an automatic Test Data Builder
tags: unittesting c# patterns autofixture
---

## The Problem
This is largely the same problem as the [Test Data Builder](http://www.garethrepton.com/Reduce-unit-test-friction-patterns-4-Test-Data-Builder/), you have non trivial test data that needs repeated construction in many tests. I'm going to modify the example here though so it fits the use case of autofixture a little better:

   {% highlight CSharp %}
 
        [Fact]
        public void GivenMeatFeedsAllLions()
        {
            //Arrange
            var zoo = new Zoo();
            zoo.Animals.Add(new Lion(name: "Leo"));
            zoo.Animals.Add(new Lion(name: "Graham"));
            zoo.Animals.Add(new Lion(name: "Jeff"));
            zoo.Animals.Add(new Rabbit(name: "Roger"));
            zoo.Animals.Add(new Rabbit(name: "Peter"));
            zoo.Animals.Add(new Rabbit(name: "Fred"));

            //Act
            zoo.Feed("Meat");

            //Assert
            var lions = zoo.Animals.OfType<Lion>();
            var rabbits = zoo.Animals.OfRabbit<Lion>();
            foreach (var lion in lions)
                lion.IsFed.Should().BeTrue();
            foreach (var rabbit in rabbits)
                rabbit.IsFed.Should().BeFalse();
        }

   {% endhighlight %}

So we have a zoo, which houses animals, each animal is its own type, but each one also has a Name field which must be passed to the constructor which we don't care about in this particular test. Each one implements a Feed and IsFed method which I'll leave to the readers imagination as to how they work.

Now.. the test data builder solution is to create a builder to construct your test objects, so your test now looks like :

{% highlight CSharp %}
            [Fact]
            public void GivenMeatFeedsAllLions()
            {
                //Arrange
                var zoo = new ZooBuilder().WithLions("Test", 3).WithRabbits("Roger",3).Build();
                //Act
                zoo.Feed("Meat");
                //Assert
                var lions = zoo.Animals.OfType<Lion>();
                var rabbits = zoo.Animals.OfRabbit<Lion>();
                foreach (var lion in lions)
                    lion.IsFed.Should().BeTrue();
                foreach (var rabbit in rabbits)
                    rabbit.IsFed.Should().BeFalse();
            }
   {% endhighlight %}

We choose not to bother setting the name in the builder, and just have it create 3 of each type of animal, then the rest of the test continues as the previous test did. In this case, the builder requires less code in the test, and is more readable, but will take a lot more code to create the builder itself and unless you have a complex object model, the trade off isn't always worth it.

## The Autofixture Solution
This is where a test object creation tool like [Autofixture](https://github.com/AutoFixture/AutoFixture) comes in extremely handy. Basically it is a generic test data builder, so you can tell it to create many strings, give it a number to create, and it will use internal (customisable) algorithms to create that number pseudo random strings e.g.

{% highlight CSharp %}
var strings = new Fixture().CreateMany<string>(20);
{% endhighlight %}

This will create a list of 20 strings, each populated according to its current configuration. This really reduces the amount of setup code required for this particular test data. And it can be extended too, so you can create much larger test models usually quite easily, to take our ZooBuilder example above the equivalent:

{% highlight CSharp %}
    [Fact]
    public void GivenMeatFeedsAllLions()
    {
        //Arrange
        var fixture = new Fixture();

        var zoo = new Zoo();
        zoo.Animals.AddRange(fixture.CreateMany<Lion>(3));
        zoo.Animals.AddRange(fixture.CreateMany<Rabbit>(3));
        //Act
        zoo.Feed("Meat");
        //Assert
        var lions = zoo.Animals.OfType<Lion>();
        var rabbits = zoo.Animals.OfRabbit<Lion>();
        foreach (var lion in lions)
            lion.IsFed.Should().BeTrue();
        foreach (var rabbit in rabbits)
            rabbit.IsFed.Should().BeFalse();
    }
{% endhighlight %}

This will generate 3 lions with pseudo random names and 3 rabbits with the same, the names will automatically get piped into each new objects constructor, and we effectively don't need to worry about setting the name field that we're not testing for. Theres slightly more code in the test than with a dedicated builder object, but we do away with maintaining a separate builder object completely. This makes autofixture incredibly powerful as a test object builder, and you can in many cases create your model objects without having to worry about the parameters you are not explicitly testing for.

## Summary
AutoFixture functions as an incredibly powerful automatic test data builder, if used sparingly it can really improve your test code by reducing the amount you have to write



