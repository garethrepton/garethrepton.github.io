---
layout: post
title: .Net Async Await, the Good, the Bad and the Deadlocks
tags: design c# software-engineering process
---

Heres a distillation of a bunch of disparate information on async await in .Net, hopefully this will work as a fairly decent primer for the core async await functionality.

This all assumes a standard SynchronisationContext (i.e. a UI context, or webapi controller context).

# What is it?
Async Await is a way of preventing blocking processes holding up an application. It works by adding some syntactic sugar onto C# with the keywords 'Async' and 'Await' and uses the Task Parallel Library under the hood to do it.

Heres a simple async method call and method:

    {% highlight c# %}
        public class SpoonRepository
        {
            public async Task<Spoon> GetSpoonsAsync()
            {
                return await this.connection.QueryAsync<Spoon>("select * from spoons"); 
            }
        }
    {% endhighlight %}

- We add the Async suffix to the GetSpoons method name, this is just convention, it makes it a little easier to spot async methods (which as you should see later is very helpful).
- The 'async' keyword just means that the method is an async method, so can be awaited by callers.  
- 'Task<Spoon>' is slightly more interesting, this tells the caller that the method doesn't return a Spoon, but it returns a Task capable of producing a spoon. 
- We use the 'await' keyword to denote a point in the method where the execution can be yielded to something that isn't blocking (whilst waiting for a blocking task)... 
- The task we are awaiting here is a database call (using Dapper) which will block.

So in short, this method will run to the await, the QueryAsync method will start the select query and return a task that will eventually return the spoons, and our method execution will yield to something else that isn't currently blocked.

We could call this method either like this:
    {% highlight c# %}
        var spoons = spoonRepository.GetSpoonsAsync().Result; //.Wait() if you don't have any result
    {% endhighlight %}
This will return the result immediately and work in a blocking manner.

Or you can call it like this from *inside* an async method:
    {% highlight c# %}
        var spoons = await spoonRepository.GetSpoonsAsync();
    {% endhighlight %}
This will yield execution further up the stack when this blocks during the data access lower down.

Some important things to note:
- You have to implement async await all the down the object model for it to work effectively.
- You shouldn't mix awaits with the .Result syntax, this can cause issues (see The Deadlocks below)
- You should return 'Task' rather than 'void' from async methods that don't return anything.

Thats pretty much all there is to the basic syntax and its actually quite self explanatory, if a bit of a leaky abstraction.

# Isn't that just like Multi-threading or Tasks?
The answer to this is sort of...  

Its important to know that this doesn't work like a standard multi threaded application would. If you have 5 tasks you'd like not to block each other in a traditional threading model, you may spin up 5 threads. In the async model, it all works from yielding the execution back to something that isn't blocked. There isn't a thread sat there waiting for the blocking operation to complete (although it may use a separate thread to execute the completion of an awaited method call).

Importantly this works the other way around too. Just because you await 5 async methods that do heavy processing, does not mean they will all execute in parallel. Unless they block, or you explicitly fire off tasks, they are likely to run synchronously. So don't expect to use async await for parallelising complex non-blocking tasks. I think this behaviour can be modified depending on the SynchronisationContext, although for this kind of workload TPL stuff is probably a less confusing and better fit overall.

Also see the Stephen Cleary [article](https://blog.stephencleary.com/2013/11/there-is-no-thread.html) for quite a low level view of how it works, hes created some really good posts on this topic.

# Always return a task
Avoid returning 'void' where possible and instead return 'Task'. Not doing this can mean exceptions bypass your exception logic and may kill your applications main thread or be caught by top level error handlers. It also means you can't call .Wait or .Result on the method to run it synchronously. 

This is exasperated when calling from inside a multi-threaded application, and having the async method call thats buried inside some thread somewhere error, and just take down the application.

# Its very easy to create Deadlocks
As I said earlier... never mix the blocking syntax and the await syntax. If you await something that calls .Result inside it, you are likely to get a deadlock depending on the context you are in, and these ones are hard to find.
Stephen Cleary offers a good example of the deadlock scenario in [Don't Block on Async Code](https://blog.stephencleary.com/2012/07/dont-block-on-async-code.html).

# Async code runs in the context its called from
So by default if its called from the UI its the UI context, or asp its the asp context. You can take it out of that context by appending:
    {% highlight c# %}
        var spoons = await spoonRepository.GetSpoonsAsync().ConfigureAwait(false);
    {% endhighlight %}
This puts it onto the ThreadPool context, and avoids the common deadlock scenario mentioned above.

# Console apps don't have a default async context
By default, console apps do not have an async context, so you have to give it one. You can't make the main method asynchronous, so you have to use something like Task.WaitAll() or something like StephenCleary's [AsyncContext](https://github.com/StephenCleary/AsyncEx/wiki/AsyncContext) to launch async methods within the console app.

# The Good
 - On the face of things, and for fairly simple cases, async await provides a fantastically simple solution to a traditionally complex problem (preventing blocking in apps).  Its far far simpler than managing threads, and locks yourself;
 - It becomes fairly easy to keep your UI responsive without requiring Background workers, explicit tasks, or threads.
 - It can reduce the number of threads required for an application to run (especially applicable to web servers).

# The Bad
 - It leaks, EVERYWHERE. You have to propagate this thing right down from the UI to the DataAccess calls to use it as intended. For some reason this really grates on me.
 - There are hidden complexities, like what happens when you forget the await keyword on one await call (Chaos)? 
 - Some thread management is still required, like switching back to the UI context in a wpf app.
 - Its not threading, but people will try to use it like it is.
 - Error messages just got a whole lot more difficult, you often end up with "A task was cancelled" instead of the error you actually want.

# In Conclusion
Async await is a fantastic tool to prevent blocking calls blocking threads and keep an application responsive in .Net. However, as with most things, it requires much deeper knowledge than just the syntax to not get into trouble with it, it doesn't work how most people first think it will work. It doesn't really fit well in all applications, for example, it would probably just complicate a necessarily multi threaded application, but could still be used if you had a use case for it.

# Further reading
See also.. there are lots of excellent articles on async await, several written by Stephen Cleary, but also many from other Authors.. See:

 - [Stephen Cleary: async and await](https://blog.stephencleary.com/2012/02/async-and-await.html)

 - [Microsoft: async](https://docs.microsoft.com/en-us/dotnet/csharp/async)

 - [Stephen Cleary: async best practices](https://msdn.microsoft.com/en-us/magazine/jj991977.aspx)

 - [Stephen Cleary: async console apps](https://blogs.msdn.microsoft.com/pfxteam/2012/01/20/await-synchronizationcontext-and-console-apps/)





