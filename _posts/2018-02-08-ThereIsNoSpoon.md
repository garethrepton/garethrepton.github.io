---
layout: post
title: There is no spoon... repository - Mocking in C#
tags: tdd design c# software-engineering process mocking
---
What do you do when you need to test a class that accepts a non-trivial dependency? Thats right, you just give up and don't test it... 

Not really, you make sure that dependency is properly abstracted in the design and replace it in the test with a Test Double. And most of the time, the test double of choice is a ***Cough*** Nusbstitute ***Cough*** Mock object... 

![A spoon]({{ "/images/spoon.jpeg" | absolute_url }})
## The Theory
Mock objects are pretty simple in theory, you have a subject under test which takes a dependency, for the purpose of the test you don't want to introduce the complexity of the dependency, so you replace it with another object that just does enough to allow the calling class to perform the function you are testing. This allows the test to remain simple, and encourages a nicely decoupled design.

## Lets set the scene
Heres our basic code:

    {% highlight c# %}

    public class CutleryProcessor
    {
        private ISpoonRepository spoonRepository;

        public CutleryProcessor(ISpoonRepository spoonRepository)
        {
            this.spoonRepository = spoonRepository;
        }

        public Dictionary<string, int> GetAvailableSpoonTypes()
        {
            return spoonRepository.GetAll().GroupBy(x => x.Type).ToDictionary(x => x.Key, x => x.Count());
        }
    }

    public interface ISpoonRepository
    {
        IEnumerable<Spoon> GetAll();
    }

    public class SpoonRepository : ISpoonRepository
    {
        public IEnumerable<Spoon> GetAll()
        {
            var connection = new SqlConnection("SomeConnectionString");
            return connection.Query<Spoon>("select * from spoons");
        }
    }

    public class Spoon
    {
        public string Type { get; set; }
    }

    {% endhighlight %}

This is a pretty basic contrived setup, you have a method on CutleryProcessor that uses the spoon repository to get available spoons, then does something with them to return a result. We want to test this "GetAvailableSpoonTypes" method.

## The problem of the spoon repository
The spoon repository here presents a problem, you could write the test like this:

    {% highlight c# %}


  public class CutleryProcessorTest
    {
        [Fact]
        public void ReturnsAvailableSpoonTypesFromRepository()
        {
            //arrange
            var repository = new SpoonRepository();
            var target = new CutleryProcessor(repository);
            //act
            var result = target.GetAvailableSpoonTypes();
            //assert
            //Lets assume 3 spoons for now
            result.Count().Should().Be(3);
        }
    }

        {% endhighlight %}


Unfortunately this means you are at the mercy of the database that the spoon repository is connecting to in this case. This is bad for many reasons:

1. Its complicated to set the data up and tear it down.
2. The data is not isolated to this test alone.
3. Without the database online, the test just won't work.
4. There are probably more reasons than this, but that will do...

Step in the mock.

## Lets replace the repository with a mock
So.. we can rewrite this test with an NSubstitute mock pretty easily like so..

    {% highlight c# %}


  public class CutleryProcessorTest
    {
        [Fact]
        public void ReturnsAvailableSpoonTypesFromRepository()
        {
            //arrange
            var repository = Substitute.For<ISpoonRepository>();

            var spoons = new []
            {
                new Spoon("dessert"),
                new Spoon("plastic"),
                new Spoon("dessert"),
                new Spoon("spork")
            };
            
            //This is the important bit...
            repository.GetAll().Returns(spoons);


            var target = new CutleryProcessor(repository);
            //act
            var result = target.GetAvailableSpoonTypes();
            //assert
            result.Count().Should().Be(3);
        }
    }

        {% endhighlight %}

In one relatively simple step, we've ridden our tests of the data access nonsense, and allowed ourselves to be able to really easily specify the test data, and expected results... 

## Summary
Thats it... You can mock dependencies this way for all kinds of things. You can also create other types of test doubles, or even just use the underlying object if its well defined and doesn't introduce complexity, or logically part of that unit being tested.












