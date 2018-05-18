---
layout: post
title:  Dapper, Deferred execution, IQueryable, Async, ToListAsync and the buffered parameter
tags: unittesting c# patterns autofixture dapper dapperdotnet .Net
---

## Intro
[Dapper](http://dapper-tutorial.net/dapper) is an awesome microorm and generally works the way I'd expect it to, and very quickly, but I fell into a little rabbit hole recently when using Dapper with async in C#, and I thought I'd document what I found out in a mini-post.

## tl;dr
When using the {% highlight CSharp %} QueryAsync<T>() {endhighlight %} extension method in Dapper, the default is to set buffered = true which executes the query asynchronously into a list as part of the method call, and means you don't need to worry about asynchronously streaming the results back to the client.

## My Question
If I write the following code:

     {% highlight CSharp %}
        var result = await connection.QueryAsync<int>("some query");
        var list = result.ToList();
     {% endhighlight %}

Dapper returns an IEnumerable<int> (it doesn't support IQueryable), which does not support ToListAsync(). So will deferred execution of the query mean that the query will execute asynchronously, but the results stream the back to the client synchronously (thanks to ToList() being a synchronous method)?

## The slightly more detailed answer
The answer to this is no. Basically dapper queries include an optional [buffered](http://dapper-tutorial.net/buffered) parameter on their interface which denotes whether the query should load data into a list within the method call, and prevents result streaming through deferred execution when the results are subsequently used. Whats more, the default value for buffered is true, so by default all async dapper queries will execute asynchronously.

If you set the buffered to false, you will get the opposite effect, the results will stream back synchronously when they are iterated and negate most of the use of making the query async (unless your results are faster to read out than your query is to execute). 

This is something to be aware of if you are using Dapper with huge resultsets too as it will mean all that data is streamed into memory at once by default, and you need to set the query so it is not buffered if you wish to make it work in a large dataset friendly way.

