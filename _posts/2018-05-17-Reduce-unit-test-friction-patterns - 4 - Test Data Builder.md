---
layout: post
title: Reduce unit test friction patterns - 4 - Test Data Builder
tags: unittesting c# patterns autofixture
---

##Intro
This was blogged about all the way back in 2007 by Nat Price [here](http://www.natpryce.com/articles/000714.html).

## The Problem
You have non trivial test data that you need to construct in order to perform your unit tests. Without any type of solution, you can quite easily end up repeating yourself in each test, and creating complicated test construction code in lots of places. 

A simple example is: 

   {% highlight CSharp %}
 public class Zoo
    {
        public List<Animal> Animals { get; set; }

        public void Feed(string food)
        {
            //Use your imagination here...
        }
    }

    public class Animal
    {
        public Guid Id { get; set; } = new Guid();
        public string Name { get; set; }

        public void Feed(string food)
        {
            IsFed = true;
        }

        public bool IsFed { get; set; } = false;
    }
        public class ZooTests
        {
            [Fact]
            public void GivenMeatFeedsAllLions()
            {
                //Arrange
                var zoo = new Zoo();
                zoo.Animals.Add(new Animal() {Name = "Lion"});
                zoo.Animals.Add(new Animal() {Name = "Lion"});
                zoo.Animals.Add(new Animal() {Name = "Lion"});
                zoo.Animals.Add(new Animal() {Name = "Rabbit"});
                zoo.Animals.Add(new Animal() {Name = "Rabbit"});
                zoo.Animals.Add(new Animal() {Name = "Rabbit"});

                //Act
                zoo.Feed("Meat");

                //Assert
                var lions = zoo.Animals.Where(x => x.Name == "Lion");
                var rabbits = zoo.Animals.Where(x => x.Name == "Rabbit");
                foreach (var lion in lions)
                    lion.IsFed.Should().BeTrue();
                foreach (var rabbit in rabbits)
                    rabbit.IsFed.Should().BeFalse();
            }

            [Fact]
            public void GivenCarrotsFeedsAllRabbits()
            {
                //Arrange
                var zoo = new Zoo();
                zoo.Animals.Add(new Animal() {Name = "Lion"});
                zoo.Animals.Add(new Animal() {Name = "Lion"});
                zoo.Animals.Add(new Animal() {Name = "Lion"});
                zoo.Animals.Add(new Animal() {Name = "Rabbit"});
                zoo.Animals.Add(new Animal() {Name = "Rabbit"});
                zoo.Animals.Add(new Animal() {Name = "Rabbit"});

                //Act
                zoo.Feed("Carrot");

                //Assert
                var lions = zoo.Animals.Where(x => x.Name == "Lion");
                var rabbits = zoo.Animals.Where(x => x.Name == "Rabbit");
                foreach (var lion in lions)
                    lion.IsFed.Should().BeFalse();
                foreach (var rabbit in rabbits)
                    rabbit.IsFed.Should().BeTrue();
            }

        }
   {% endhighlight %}

You can see that even in this simple example, theres repeated object construction code going on.

## The Solution
Use a Test Data Builder to abstract away the data construction for your tests. A  Test Data Builder is a simple fluent object which is usually designed in such a way as to allow you to construct the data for your tests in a simpler, and safer way.

Heres an example of a Test Data Builder for the above example:

     {% highlight CSharp %}
    
    public class ZooBuilder
    {
        private List<Animal> Animals { get; set; } = new List<Animal>();

        public ZooBuilder WithAnimals(string name, int amount)
        {
            for (int i = 0; i < amount; i++)
                this.Animals.Add(new Animal() { Name = name });

            return this;
        }

        public Zoo Build()
        {
            var zoo = new Zoo() { Animals = this.Animals};
            return zoo;
        }
    }

     {% endhighlight %}

So a couple of important points here... 

1. To add an animals (or set of) you call the "WithAnimals" method. This returns the ZooBuilder after its appended the new animals, this is to make the api fluent so you can just do:    {% highlight CSharp %}  new ZooBuilder().WithAnimals("Lion", 3).WithAnimals("Rabbit", 3);    {% endhighlight %}. This is very powerful as it means test construction can be very clear and succinctly expresses the test setup.
2. The build method builds the test data, which in this case is actually the target of the test.

This object now massively simplifies our earlier tests and they become:
     {% highlight CSharp %}
            [Fact]
            public void GivenMeatFeedsAllLions()
            {
                //Arrange
                var zoo = new ZooBuilder().WithAnimals("Lion", 3).WithAnimals("Rabbit", 3).Build();
                //Act
                zoo.Feed("Meat");
                //Assert
                var lions = zoo.Animals.Where(x => x.Name == "Lion");
                var rabbits = zoo.Animals.Where(x => x.Name == "Rabbit");
                foreach (var lion in lions)
                    lion.IsFed.Should().BeTrue();
                foreach (var rabbit in rabbits)
                    rabbit.IsFed.Should().BeFalse();
            }

            [Fact]
            public void GivenCarrotsFeedsAllRabbits()
            {
                //Arrange
                var zoo = new ZooBuilder().WithAnimals("Lion", 3).WithAnimals("Rabbit", 3).Build();

                //Act
                zoo.Feed("Carrot");

                //Assert
                var lions = zoo.Animals.Where(x => x.Name == "Lion");
                var rabbits = zoo.Animals.Where(x => x.Name == "Rabbit");
                foreach (var lion in lions)
                    lion.IsFed.Should().BeFalse();
                foreach (var rabbit in rabbits)
                    rabbit.IsFed.Should().BeTrue();
            }
     {% endhighlight %}

So as you can see, most of the construction code for the tests is now sat behind the builder object. This has had the following effects:
1. The test code is now much more succinct and readable.
2. If the zoo objects, or animal objects change their construction interface, only the builder object will need to change, not all of our tests so our tests are less fragile.

This is a very useful pattern for more complex test setup routines, especially anything verging on an integration test. You can also have builders calling builders too to add another level of abstraction, for example I could have split the logic above into an animal builder and zoo builders separately, and had the zoo builder use animal builders to construct its animals.

## When to use
Test Data Builders thrive in more complicated tests that require a lot of setup, basically the more complicated the setup for a test object model, the more likely a Test Data Builder object will help. And conversely, the simpler the test the less likely it is to help, and maintaining it is more likely to just get in the way so don't use this for very simple tests as you can easily find yourself writing a lot of unnecessary code.
