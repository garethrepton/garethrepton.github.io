---
layout: post
title: FSharp - Category Theory - Monoid
tags: design f# currying partial-application categorytheory unit software-engineering process .Net FSharp FunctionalProgramming
---

## Introduction
This is the first post on category theory terms, which is sort of like a mathematical version of design patterns in the functional world, but as you might expect they are at a much lower level than the OO design patterns. 

One particularly weird thing to get your head around with category theory terms, is they don't apply to the data type or the function in isolation. They apply to the function over a given data type, so its worth bearing this in mind when you read about category theory.

## What is a Monoid?
The short description is:

> *A monoid is an operation on a datatype which is binary, associative and has an identity*

Now, this is one of those sentences with a heap of stuff in that makes no sense unless you define it, so lets unpack that a bit:

## Binary operation
To be a binary operation simply means takes to inputs of a set, and produces an output from the same set e.g.

{% highlight FSharp %}

let add x y = x + y
let multiply x y = x * y

{% endhighlight %}

but these are not binary operations because the output type is not of the same set as the inputs (unless the inputs are strings that is):

{% highlight FSharp %}

let addToString x y = (x + y).ToString()
let multiplyToString x y = (x * y).ToString()

{% endhighlight %}

## Associative operation
To be an associative operation means that the order of application doesn't matter. So for example adding 3 integers `(1 + 2) + 3 = 6` produces the same output written as `1 + (2 + 3) = 6` and thats true for all types of addition. The same is also true for multiplication so `4 * (2 * 3) = 24` and `(4 * 2) * 3 = 24`. 

## The Identity
Identity in category theory world means a value that does nothing (i.e. returns the original value). So for example `1 + 0 = 1` or `2 * 1 = 2`. That is literally all the identity is, its a value which when applied to the data type and function returns the original input.

## Examples of Monoids
So we've already seen the simplest and most obvious examples these are:

* Add
{% highlight FSharp %}
//definition
4 + (2 + 3) = 9
(4 + 2) + 3 = 9
(4 + 2) + 3 + 0 = 9 //Identity = 0
{% endhighlight %}

* Multiply
{% highlight FSharp %}
4 * (2 * 3) = 24
(4 * 2) * 3 = 24
(4 * 2) * 3 * 1 = 24 //Identity = 1
{% endhighlight %}

String concatenation also fits too:

* String concatenation
{% highlight FSharp %}
"A" + ("B"  + "C") = "ABC"
("A" + "B")  + "C" = "ABC"
("A" + "B")  + "C" + "" = "ABC" //Identity = ""
{% endhighlight %}

All of these comply with our above rules. And are pretty straightforward.

## Examples of things that aren't Monoids

* Subtraction
{% highlight FSharp %}
//definition
4 - (2 - 3) = 5
(4 - 2) - 3 = -1
{% endhighlight %}

Subtraction doesn't apply because it isn't associative, changing the order of the subtraction matters.

* Division
{% highlight FSharp %}
4 / (4 / 2) = 2
(4 / 2) / 4 = 0
{% endhighlight %}

Similar to subtraction, division is not associative.

## Summary
Thats just about it, after thinking about what a monoid is and how it applies to code for a bit I suspect they'll start appearing like when learning about design patterns, it takes a little while to familiarise yourself to the terminology and what to look for. Its a bit harder in this case because the concept is slightly more abstract and the terminology doesn't really imply what it means to anybody except mathematicians. 

## A few resources I've read/used
Mostly I've been listening to the category theory episodes of the lambdacast podcast to learn these techniques, but have also read the following blog posts too:
* [LambdaCast](https://soundcloud.com/lambda-cast)
* [Mark Seemann's blog](http://blog.ploeh.dk/2017/10/06/monoids/)
* [FSharp for fun and profit](https://fsharpforfunandprofit.com/posts/monoids-without-tears/)