---
layout: post
title: .Net Async Await, the good, the bad and the deadlock
tags: design c# software-engineering process
---

I'm going try to distill as much information about async/await into one post as possible.

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
- 'async' just means that the method is an async method, so can be awaited by callers.  
- 'Task<Spoon>' is slightly more interesting, this tells the caller that the method doesn't return a Spoon, but it returns a Task capable of producing a spoon. 
- We use the 'await' keyword to denote a point in the method where the execution can be yielded to something that isn't blocking (whilst waiting for a blocking task)... 
- The task we are awaiting here is a database call (using Dapper) which will block.

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
- You shouldn't mix awaits with the .Result syntax, this can cause issues.
- You should return 'Task' rather than 'void' from async methods that don't return anything.

Thats pretty much all there is to the basic syntax and its actually quite self explanatory, if a bit of a leaky abstraction.

# Isn't that just like Multi-threading or Tasks?
The answer to this is sort of...  

Its important to know that this doesn't work like a standard multi threaded application would. If you have 5 tasks you'd like not to block each other in a traditional threading model, you may spin up 5 threads. In the async model, it all works from yielding the execution back to something that isn't blocked. There isn't a thread sat there waiting for the blocking operation to complete. 

Importantly this works the other way around too. Just because you await 5 async methods that do heavy processing, does not mean they will all execute in parallel. Unless they block, or you explicitly fire off tasks, they are likely to run synchronously. So don't expect to use async await for parallelising complex non-blocking tasks.

Also see the Stephen Cleary [article](https://blog.stephencleary.com/2013/11/there-is-no-thread.html) for quite a low level view of how it works, hes done quite a few really good posts on this topic.

# Async Contexts
<TODO>

# Always return a task
Avoid returning 'void' where possible and instead return 'Task'. Not doing this can mean exceptions bypass your exception logic and may kill your applications main thread or be caught by top level error handlers. It also means you can't call .Wait or .Result on the method to run it synchronously.

# The Good
 - On the face of things, and for fairly simple cases, async await provides a fantastically simple solution to a traditionally complex problem (preventing blocking in apps).  Its far far simpler than managing threads, and locks yourself;

# The Bad
 - It leaks, everywhere. You have to propogate this thing right down from the UI to the DataAccess calls to use it as intended. For some reason this really grates on me.
 - There are hidden complexities, like what happens when you forget the await keyword on one await call (Chaos)? 
 - Some thread management is still required, like switching back to the UI context in a wpf app.
 - Its not threading, but people will try to use it like it is.
 - Error messages just got a whole lot more difficult, you often end up with "A task was cancelled" instead of the error you actually want.

# The Deadlocks
As I said earlier... never mix the blocking syntax and the await syntax. If you await something that calls .Result inside it, you are likely to get a deadlock depending on the context you are in.
<TODO>


#Further reading
See also.. there are lots of excelle

https://docs.microsoft.com/en-us/dotnet/csharp/async


https://msdn.microsoft.com/en-us/magazine/jj991977.aspx

https://blogs.msdn.microsoft.com/pfxteam/2012/01/20/await-synchronizationcontext-and-console-apps/





