---
layout: post
title: FSharp - Currying and Partial Application |>  ft w
tags: design f# currying partial-application unit software-engineering process .Net FSharp FunctionalProgramming
---

## Intro
A little bit of magic from the functional world...

Read the following 2 examples closely.

This produces the number 3 in F#:

{% highlight FSharp %}
        let add x y = x + y
        let r = add 1 2;
{% endhighlight %}
_Example 1_

This next code also produces the number 3 in F#:

{% highlight FSharp %}
    let add x y = x + y
    let partialAdd = add 1
    let r = partialAdd 2
{% endhighlight %} 
_Example 2_ 

The second solution is using something called partial application, and its a bit weird for us OO developers.

## What is partial application?
Partial application is the act of partially applying a function to create a new function and is allowed in functional languages because they do currying. 

### Currying 
Currying is where every function in a language is executed as though it actually has only 1 argument, so in the case of add above it actually executes like this `(Add 1) + 2` where `(Add 1)` is the first function, and `x + 2` is the second function where `x` is the result of `(Add 1)`. Each time you call a function with a parameter, you get back a new function unless you've specified all of the parameters for the function. If we supply all the arguments the first time we call the function like we do in the first example, the currying is transparent and we get the value back that is the result of the `add`, so `r = 3`, but internally the currying will still take place.

### Partial application
Currying allows for partial application of functions, so we can supply any function an incomplete number of arguments, but "bake-in" arguments that we don't want to supply each time, and we get back a new function that we can call with the reduced number of parameters. So `let partialAdd = add 1` actually just returns a new function with a single int parameter called `partialAdd` with the parameter 1 baked into it because we haven't supplied all of the arguments to complete the function call. Subsequently calling this new function with the argument 2, completes the function call and adds the baked-in 1 to 2 and returns the int value 3.

## Why is this useful?
To think about this as a C# or OO developer, it opens up a whole new level of composition that we you can't really get in non-functional languages. Its at a much lower level than we're used to, and it allows incredibly succinct composition of behaviours that we'd end up writing classes and interfaces for normally. Lets write some code and see how it helps in a simplistic logging scenario.

### A simple log example
For a simple example, lets consider a case where you want to write a performance log to the console. So after a series of events in your application you want to output roughly how long they took to execute with a uniform message. Now in C# world this looks something like this:

{% highlight CSharp %}

void Main()
{
	Log(DateTime.Now, $"First");
	Thread.Sleep(1000);
	Log(DateTime.Now, $"Second");
	Thread.Sleep(1000);
	Log(DateTime.Now, $"Third");
	Thread.Sleep(1000);
}

public static void Log(DateTime time, string message)
{
	Console.WriteLine($"{time} - {message}");
}

{% endhighlight %}  

_Printing_:
* 30/11/2018 18:30:52 - First
* 30/11/2018 18:30:53 - Second
* 30/11/2018 18:30:54 - Third

A direct conversion in F# is something like:

{% highlight FSharp %}

let sleep x =  Async.Sleep x |> Async.RunSynchronously
let log date message= printfn "%s %s" date message
let getDate = DateTime.Now.ToString()

log getDate "First"
sleep 1000
log getDate "Second"
sleep 1000
log getDate "Third"
sleep 1000 

{% endhighlight %}  

_Printing_: 
* 30/11/2018 18:34:50 first
* 30/11/2018 18:34:50 second
* 30/11/2018 18:34:50 third

OK, so far very similar really.

**Wait**... all those times are the same in the F# example? 

### Function calls are not always re-evaluated
This is something that tripped me up at first writing this, but theres a reason for it, when you call a function without arguments the value it returns is bound to the function call, it does not get re-executed. If you call a function with arguments the function is what gets bound to the variable, so each time you call this, you have to specify its parameters, and it will re-execute. Now in our case, we have no arguments, so we will need to supply the "Unit" type to the function, which means using "()" as the single argument to the getDate function, and as the first argument to the partial logg function to ensure both get re-executed on each call, and we get the right times printed to the log. 

### Partially applied version
Lets modify the F# version to use the unit parameter and partial application for the log getDate argument:

{% highlight FSharp %}

let sleep x =  Async.Sleep x |> Async.RunSynchronously
let log date message= printfn "%s %s" date message
let getDate () = DateTime.Now.ToString()
let logg () = log (getDate ())

logg () "First"
sleep 1000
logg () "Second"
sleep 1000
logg () "Third"
sleep 1000

{% endhighlight %}  

_Printing_: 
* 01/12/2018 07:12:37 First
* 01/12/2018 07:12:38 Second
* 01/12/2018 07:12:39 Third

This time things start to look a little different in a couple of ways. The first is that I've modified the `logg` function and `getDate` to take the unit parameter to make sure they re-execute each time and don't memoize their values, this is slightly less succinct, but acceptable.

The other major difference is that I've made the call to log slightly more succinct by creating a new function called logg which is the result of partially applying log with just the getDate part. This same effect could be achieved in C# with an optional parameter on the date argument, and an if statement to set it, but it takes more code, so the F# version is more succinct even in this really simple example. Personally I think the F# version looks much better in this case.

### A more in-depth example
Lets make this more complicated and see where it goes, lets make the log method capable of outputting to multiple places (simulating a real logging system like Log4Net etc...). 

{% highlight CSharp %}

void Main()
{
	var logger = new Logger(new ILogWriter[] { new ConsoleLogWriter(), new FakeSqlLogWriter()});

	logger.Log("Tester", DateTime.Now, $"First");
	Thread.Sleep(1000);
	logger.Log("Tester", DateTime.Now, $"Second");
	Thread.Sleep(1000);
	logger.Log("Tester", DateTime.Now, $"Third");
	Thread.Sleep(1000);
}

public static void Log(DateTime time, string message)
{
	Console.WriteLine($"{time} - {message}");
}

// Define other methods and classes here
public class Logger
{
	public IEnumerable<ILogWriter>  Writers { get; set; }

	public Logger(params ILogWriter[] writers)
	{
		this.Writers = writers;
	}

	public void Log(object sender, DateTime date, string message)
	{
		foreach (var writer in Writers)
			writer.Output(sender, date.ToString(), message);
	}
}

public interface ILogWriter
{
	void Output(object sender, string time, string message);
}

public class ConsoleLogWriter : ILogWriter
{
	public void Output(object sender, string time, string message)
	{
		Console.WriteLine($"{sender} - {time} - {message}");
	}
}

public class FakeSqlLogWriter : ILogWriter
{
	public void Output(object sender, string time, string message)
	{
		Console.WriteLine($"IAMSQL: {sender} - {time} - {message}");
	}
}

{% endhighlight %}  

**54 lines of code**

_Printing_: 
* Tester - 30/11/2018 19:02:41 - First
* IAMSQL: Tester - 30/11/2018 19:02:41 - First
* Tester - 30/11/2018 19:02:42 - Second
* IAMSQL: Tester - 30/11/2018 19:02:42 - Second
* Tester - 30/11/2018 19:02:43 - Third
* IAMSQL: Tester - 30/11/2018 19:02:43 - Third

Here I've added a sender argument to our log method, moved it into an object, and added an interface for writing the various types of log output. This is pretty typical of a logger implementation in c# world, I've added and instantiated 2 writers, the first is our console writer, the second is a fake sql writer which just writes to the console anyway with "IAMSQL" at the start of the message. Quite a lot of code but this is a fairly solid object oriented solution.

Heres the same behaviour in F#:

{% highlight FSharp %}

let sleep x =  Async.Sleep x |> Async.RunSynchronously
let logConsole  sender date prefix message= printfn "%s%s %s %s" prefix sender date message
let getDate () = DateTime.Now.ToString()
let logConsolePartial () = logConsole "Tester" (getDate())

let logActualConsole () = logConsolePartial () "" 
let logFakeSql () = logConsolePartial () "IAMSQL "
let logWriters = [logActualConsole; logFakeSql]

let log message = logWriters |> Seq.iter(fun x ->   x () message)

log "First"
sleep 1000
log "Second"
sleep 1000
log "Third"
sleep 1000

{% endhighlight %}  

**17 lines of code**.

_Printing_:
* Tester 01/12/2018 07:19:48 First
* IAMSQL Tester 01/12/2018 07:19:48 First
* Tester 01/12/2018 07:19:49 Second
* IAMSQL Tester 01/12/2018 07:19:49 Second
* Tester 01/12/2018 07:19:50 Third
* IAMSQL Tester 01/12/2018 07:19:50 Third

I'd say thats a pretty good example of how partial application can be beneficial. So instead of using the polymorphic approach we used in C#. Our LogWriters become a list of partially applied functions instead of objects. 

1. logConsole is our initial function that actually logs to the console, and accepts a prefix as well as the standard arguments (so we can fake the Sql Log Writer).
2. logConsolePartial is an intermediate step, we call logConsole partially with the common arguments we're going to use throughout this code. In this case, the sender, and the getDate function.
3. logActualConsole is then a partial function call to logConsolePartial with no prefix.
4. logFakeSql is also a partial function call to logConsolePartial with the IAMSQL prefix.
5. logWriters is then our list of loggers;
6. finally log takes in the message parameter then pipes the list of loggers into an iterator, which calls each in turn with the message passed in.

Really what we've done here is replaced the strategy pattern we used for the set of ILogWriter's in C# with a pair of "Strategy" functions. So `FakeSqlLogWriter` becomes the function `logFakeSql` and `ConsoleLogWriter` becomes `logActualConsole`. We've also replaced the dependency injection with our log function, which takes in the partially applied logFakeSql and logActualConsole functions, and completes their call with the message parameter. **Its like we're doing object composition, but its a level lower at the function level, so the "Framework" code required to plumb it all together is massively reduced**. We're composing functions together here with minimal code, and fuss (except that unit parameter), and its much much more succinct than the c# version, but it still expresses what its doing effectively, probably more effectively than in C# as theres less code to read. 

## Summary
Partial application is a functional concept thats made possible because F# uses currying to make sure that all functions actually have a single parameter. Its very powerful (I'm sure there are many weird and wonderful uses for it I've not even considered yet), but with this power comes a little extra complexity, I can see the unit parameter being a bit like the deferred execution issues you get with Developers new to Linq. It is yet another tool in F# that allows for writing succinct, yet readable code, something thats very hard to do in C#, especially using pure object oriented features.

## A few resources I've read/used
This started as a simple post about partial application, but turned into a bit more so I've read quite a lot to get to this point, here are a few of the resources I've used to learn about the concepts:

[LambdaCast (partial application episode)](https://soundcloud.com/lambda-cast)
[Currying](https://fsharpforfunandprofit.com/posts/currying/)
[Partial Application](https://fsharpforfunandprofit.com/posts/partial-application/)
[Information about the unit parameter](https://stackoverflow.com/questions/17870937/what-does-this-notation-mean/17872004#17872004)
[Linqpad](https://www.linqpad.net/)
