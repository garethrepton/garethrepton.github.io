---
layout: post
title: FSharp - Call me maybe, or option
tags: design f# software-engineering process .Net FSharp FunctionalProgramming
---

## Intro
The steady march towards learning F# continues. This time we need to talk about the option type, what it is, how it compares to nullables in C# and the basics of how you are supposed to deal with it. 

The closest relative in the C# world is the `Nullable<T>` type. Which allows you to declare something thats not assignable to Null (e.g. an `int`) as nullable, and has properties that declare whether the variable is assigned a value or not, and gives you access to the value. For example:

{% highlight CSharp %}
	int? test = null;

	if (test.HasValue)
		Console.WriteLine($"{test.Value}");
{% endhighlight %}
> Nothing is printed

This useless piece of code will do absolutely nothing, because it declares the `Nullable<int>` "test" with the shorthand `?` operator. It then proceeds not to assign anything to test, and to check its `HasValue` property to see if its `Null`. If it wasn't `Null` it would then print its value to the screen. Now we could of course not do this null check like so:

{% highlight CSharp %}
	int? test = null;

	Console.WriteLine($"{test.Value}");
{% endhighlight %}
> *System.InvalidOperationException: 'Nullable object must have a value.'*

Of course, this code will throw an exception, which is one step further than the first code that did nothing. Finally, just for completeness:

{% highlight CSharp %}
	int? test = 1;

	Console.WriteLine($"{test.Value}");
{% endhighlight %}
> 1

This prints the value 1, because thats whats been assigned to the nullable int.

## Hey, I just met you, and this is crazy, but whats this got to do with F#?
Well, functional languages don't tend to allow Nulls, instead they deal with something called a `maybe` or in F# (and OCAML) world `option`. In F# you declare something as an option type by declaring it with the word `Some`. For example:

{% highlight FSharp %}
  	let x = Some 41;
    x |> string |> fun x -> printfn "%s" x
{% endhighlight %}
> Some(41)

What this piece of code is effectively saying is that x may contain an integer value of 41, BUT consumers of this variable need to make sure they handle the case where it doesn't exist. Fortunately for our example case above, theres an automatic conversion to a string for the option type and we got "Some(41)" printed to the console. 

Just to show the other case:

{% highlight FSharp %}
  	let x = None;
    x |> string |> fun x -> printfn "%s" x
{% endhighlight %}
> Prints nothing

We've assigned None, and we got None.

Immediately its fairly clear that these are a different beast to the Nullable types in C# in many ways, part of which is that the compiler will actually check them. So to show this, lets try adding 1 to our x variable as it is:

![failstocompile]({{ "/images/FSharpOptions/2019-01-28_18-51-12.png" | absolute_url }})

As you can see, unlike in C# where you could just try to use .Value and get a runtime exception, with option we get a compile time error if we fail to handle an optional type. So how should we handle an optional type then?

## In steps pattern matching
One of the really powerful features in F# is pattern matching. Its pretty useful for handling options, as its basically a really concise switch statement. Lets finish our add function with a pattern matched function to handle the option.

{% highlight FSharp %}
    let x = Some 41;

    let addOption x y : int  = match x with
        | None -> 0 + y
        | Some x -> x + y

    let y = addOption x 1;
    y |> string |> fun x -> printfn "%s" x
{% endhighlight %}
> 42

This time, we've handled that option type by creating a function called `addOption`, all that function does is decide how to handle the logic of adding the two numbers together, either with a default value for the `None` case, or with the actual value otherwise. The pattern matching syntax looks a little weird at first, but you get used to it, its basically `match <variable> with |` where `|` is the separator for each case. Within the first case we simply match x with `None`, this is our condition where x isn't set, so we declare it as `0 + y`. In our second case, we say `Some x` which means x is set, and then we can just use x as a standard integer. 

This looks like quite a lot more code than we had for our C# example above, but the important thing to remember is that the compiler has made you handle the situation, you won't get a nasty runtime exception surprise here. And because its functional, it can be refactored fairly easily to make this more generally applicable. Lets assume sometimes we want to multiply our numbers as well. We ideally want to reuse our option logic by passing in a function for the operator, and specifying the default value for our particular action.

{% highlight FSharp %}
    let x = Some 41;
    
    let operateOnOption def func x y : int  = match x with
        | None -> func def y
        | Some x -> func x y
        
    let add = operateOnOption 0 (+)
    let multiply = operateOnOption 1 (*)
    
    add x 1
        |> string |> fun x -> printfn "%s" x
    
    multiply x 2 
        |> string |> fun x -> printfn "%s" x
{% endhighlight %}
> 42
> 82

I know its a slight tangent, but its fun and hopefully its a bit clearer that this sort of explicit handling isn't as much of a chore than it looked like after the previous example. So in this case I've changed our function to be called `operateOnOption` and added two arguments `def` which takes our default for when our option is `None` and func which is our function to run against the two items. I then partially apply the function to create add, and multiply functions, before applying it to our existing add, and a multiplication of times 2.

## Nothing is nullable
