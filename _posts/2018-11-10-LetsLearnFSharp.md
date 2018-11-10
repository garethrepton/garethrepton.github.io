---
layout: post
title: Day 1 of functional programming for the object oriented inclined
tags: design f# software-engineering process .Net FSharp FunctionalProgramming
---

## lets(learn(fsharp("")))

     {% highlight FSharp %}

        let lets x = x
        let learn x = x
        let fsharp x = "OK"

        let res = lets(learn(fsharp("")))
        res.Dump();

     {% endhighlight %}

![My first f# program]({{ "/images/fsharp1.png" | absolute_url }})

## Intro
So I've decided to learn a bit more about functional programming, the more I hear about it the more I quite like it. I've been listening to the excellent [LambdaCast](https://twitter.com/lambdacast) and reading some of the articles on [FsharpForFunAndProfit](https://fsharpforfunandprofit.com/) and its clear there appears to be some merit to the functional mindset. I'm also quite interested to understand if I'm already using a certain amount of the functional mindset in my c# code.

I'll probably write the interesting tidbits I spot along the way, as there are plenty of F# tutorials out there.

## Day 1
Its day 1, what do you do, apart from a sarcastic heading and supporting program for a blog post? Lets write something a bit fruity in c# and in f# and compare them... lets write everyones favourite sample code question, [FizzBuzz](http://wiki.c2.com/?FizzBuzzTest) (yay!). Today I'm using [linqpad](https://www.linqpad.net/) with the trendy dark mode skin, I might go full hipster and switch to VSCode in the future though.

So the spec (borrowed from the c2 wiki):

> "Write a program that prints the numbers from 1 to 100. 
> But for multiples of three print “Fizz” instead of the number 
> and for the multiples of five print “Buzz”.
> For numbers which are multiples of both three and five print “FizzBuzz”."

### C# Solution:

     {% highlight CSharp %}
        void Main()
        {
            for(var i = 1; i <= 100; i++)
            {
                FizzItUp(i).Dump();
            }
        }

        public static string FizzItUp(int i)
        {
                if (i % 15 == 0)
                    return "FizzBuzz";
                else if(i % 3 == 0)
                    return "Fizz";
                else if (i % 5 == 0)
                    return "Buzz";
                else
                    return i.ToString();
        }
     {% endhighlight %}

### F# solution (attempt 1):

Heres where I quickly realised I know barely any syntax... but bear with me :).

     {% highlight FSharp %}
        let fizzItUp i = 
            if i % 15 = 0 then "FizzBuzz"
            elif i % 3 = 0 then "Fizz"
            elif i % 5 = 0 then "Buzz"
            else i.ToString()

        for i = 1 to 100 do
            let x = fizzItUp i;
            x.Dump();

     {% endhighlight %}

Ok.. so this WORKS and is really succinct and nice I thought, then I read [this](https://fsharpforfunandprofit.com/posts/control-flow-expressions/) which says for loops and if then else are imperative and not functional concepts, and as the goal here is to learn functional programming, its immediately time for a rewrite...

### F# solution (attempt 2):

     {% highlight FSharp %}

        let fizzItUp i = 
            match i % 15 with 0 -> "FizzBuzz" | _ when i % 3 = 0 -> "Fizz" | _ when i % 5 = 0 -> "Buzz" | _ -> i.ToString()

        let res = [1..100] |> Seq.map(fizzItUp)
        res.Dump()

     {% endhighlight %}

This works, and is more declarative than my first attempt, which was than imperative, which is a more functional way of thinking.

### Observations

Look at the succinctness in the final F# example!, its an elegantly small amount of code. Whilst I'm not entirely sure the match feature is that much better than the if in this particular case, I can see how it would be advantageous for things where you have a fixed map between arguments (sort of like a c# switch statement). The sequence logic is much simpler than a for loop, and less verbose and self explanatory, which is also nice. 

Its definitely more declerative rather than imperative, you wouldn't necessarily want to step through this, you'd read it to check it is correct, then run it to test the outcome.

Types are being inferred, theres no specifying them anyway, kind of like var in C# but without even needing that, but there is some control there, these are not dynamic types at all, when I tried to return i without the ToString() from the match statement, there was a compile error stating there was a type mismatch from int to string.

The map function kind of reminds me of lambda statements in c#, it will be interesting to see if parralels continue between F# and the C# linq (lambda) statements going forwards.







