---
layout: post
title: FSharp - Thinking a bit more functionally for a prime number calculator
tags: design f# software-engineering process .Net FSharp FunctionalProgramming
---

## Intro
For my second post on F# I'm sticking with simple examples, and I'm going to write a really basic prime number calculator. Again I'm writing this in [linqpad](https://www.linqpad.net/) as an "F# program".

So what is a prime number?

> "A prime number that is divisible only by itself and 1."

A nice simple example then... this is not going to try to be a smart mathematical solution to this, its going to be a brute force approach because it will take more code to write, and I'll have to iterate etc...

## The Solution
First of all, a few rules I've picked up on F# from reading various sources (primarily the brilliant [FSharpForFunAndProfit](https://fsharpforfunandprofit.com)) and experimenting this week:

* **Everything is a function** - Seriously everything, the language sort of forces you into this mindset
* **Chaining functions is powerful** - Piping the results of one function directly to another is brilliant
* **Higher order functions are powerful** - These are functions that can operate on other functions, it means common behaviour's can be packaged together nicely, and not reused
* **Functions in F# are much, much more succinct that C#** - FSharp does not require the types of declarations that need to be done in C# to pass around functions, most stuff is inferred.
* **I still don't quite know what a Monad is** - :)

The F# solution I came up with is:

     {% highlight FSharp %}

        //1. The actual prime number calculations
        let isDivisible a b = a % b = 0
        let totalDiv x = [1..x] |> Seq.filter(fun y -> isDivisible x y) 
        let isPrime x = totalDiv x |> Seq.length = 2

        //2. Utility functions, really just to show what its doing
        let print x = printfn "%s" x
        let string x = x.ToString()
        let concat x = String.concat "\r\n" x

        //3. This is the statement that joins the functions together
        [2..10000] 
            |> Seq.map(fun x -> x, isPrime x) 
            |> Seq.map(fun x -> string x ) 
            |> concat 
            |> print

     {% endhighlight %}

This outputs:

     {% highlight PlainText %}
        2, True)
        (3, True)
        (4, False)
        (5, True)
        (6, False)
        (7, True)
        (8, False)
        (9, False)
        (10, False)
        (11, True)
        (12, False)
        (13, True)
        (14, False)
        (15, False)
        (16, False)
        (17, True)
        (18, False)
        (19, True)
        (20, False)
        (21, False)
        ...
     {% endhighlight %}

1. We declare our 3 functions that we need to check if a single number is prime.
    1. isDivisible = just checks that dividing 1 number against another has 0 remainder
    2. totalDiv = this one is the functional equivalent of a for loop, it iterates the numbers between 1 and x, and returns those that are divisible without remainder.
    3. isPrime = this calls totalDiv, and checks the result has a count of 2, if it does, by definition it can only be divisible by 1 and itself so is prime.
2. These are our helper functions effectively, we could write this without those, but it just makes the code a bit clearer
3. This joins our functions together to calculate all prime numbers between 2 and 10000 by:
    1. Generating a list of numbers in the range we care about
    2. Piping that to the map function, that calls isPrime and gives us a result
    3. Piping the results of that to a map function that converts our result to a string.
    4. Piping that to concat (the argument is inferred by the pipe)
    5. Piping that to print (the argument is inferred by the pipe)

## Observations
There are some fairly interesting things emerging from this code. EVERYTHING is really a function, and you are encouraged to write very small functions, this leads to a world where everything is also composable. This is a very powerful difference between Functional and OO paradigms, where OO encourages composition in the form of objects, F# encourages composition in the form of functions. You can do some of this in C# by passing around Func<T..>, it just never quite feels right in the language, it always feels like a bit of a hack. Whereas in F#, everything is a Func, and it makes the passing around of functions more natural, but also much more readable than C#. I'm not sure how this looks in a larger application yet, but this also means you could probably write a function once, and use everywhere in the application, especially higher order functions whose argument types could vary with the functions that are passed into them. You can also do this in C# but there are static types, generics, and type arguments that need to be specified, that just feel a bit like they get in the way of it.

Another interesting thing here is that 3. actually just looks a whole lot like a lambda chain in C#. Thats because the Lambda functions in C# are basically a bit of functional declarative programming that C# developers can, and probably do use every day. So thats good news :).

## And finally

With these thoughts in mind, I've thought of a small refactor that can be made to make this more functional still...

Old code:
    {% highlight FSharp %}

        //1. The actual prime number calculations
        let isDivisible a b = a % b = 0
        let totalDiv x = [1..x] |> Seq.filter(fun y -> isDivisible x y) 
        let isPrime x = totalDiv x |> Seq.length = 2
    
    {% endhighlight %}

New code:
     {% highlight FSharp %}

        //1. The actual prime number calculations
        let isDivisible a b = a % b = 0
        let matches f x = [1..x] |> Seq.filter(fun y -> f x y) 
        let isPrime x = matches isDivisible x |> Seq.length = 2

     {% endhighlight %}

So what did I do? I changed the "totalDiv" function to take in a function, and renamed it to "matches", I also changed the way it gets called in isPrime to pass the isDivisible function. So we're then left with three very specific functions that each only care about 1 thing, and "matches" is now very general, we could pass it any function that takes two values and returns a boolean.








