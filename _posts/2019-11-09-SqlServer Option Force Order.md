---
layout: post
title: Option FORCE ORDER in SqlServer
tags: C# Sql TSql Query Tuning SqlServer visualstudio dotnet projects database query optimisation
---

## Intro
I came across Option FORCE ORDER a few months ago whilst looking into performance issues with a particular query, and thought it was worth writing down as I'd never seen it before and its quite useful.

## Why OPTION FORCE ORDER?
I had what appeared to be everything correctly lined up for an efficient query plan to be generated, we selected from the smallest data volume table at the start of the query, and then joined to a couple of larger volume tables further down. However, SqlServers query plan was picking an inefficient plan that involved pulling far more data than it needed to from the larger tables before filtering them down to the initial smaller volume tables. Ordinarily at this point I'd usually be thinking about breaking apart the query to 'trick' the query plan generator into producing a better query, or using temporary tables etc... However, I got to thinking, surely SQL Server has a query hint for this, and sure enough, there it was in the [Documentation](https://docs.microsoft.com/en-us/sql/t-sql/queries/hints-transact-sql-query?view=sql-server-ver15).

OPTION FORCE ORDER specifies that the join order of the query should be preserved during query optimisation (as specified by MSDN), this means in my case the query plan changed and generated the plan based on the smaller table, then joining to the larger tables, massively reducing the query time and load.  

So for a rough example, lets say we have the following tables:

 * Person - 100 rows
 * Order - 200,000,000 rows
 * Invoice - 200,000,000 rows

And you have the following query:

{% highlight SQL %}

Select * from Person p
inner join Order o on p.Id = o.PersonId
inner join Invoice I on o.Id = I.InvoiceId
Where I.InvoiceReference = @InvoiceReference

{% endhighlight %}

In my case, for some reason the query plan was filtering based on address, then linking to order, and then finally linking to person. Assuming the reference filter on the invoice table is more costly than filtering based on people first, this is the wrong way round (your mileage may vary here). Your query will be iterating/scanning/seeking far more rows than it needs to and performing a potentially expensive operation on them. So... what happens if we add our option to the end:

{% highlight SQL %}

Select * from Person p
inner join Order o on p.Id = o.PersonId
inner join Invoice I on o.Id = I.InvoiceId
Where I.InvoiceReference = @InvoiceReference
OPTION	(FORCE ORDER)

{% endhighlight %}

In this case, the query plan inverts from its previous incarnation, and starts with the smaller person table, filtering down the order and invoice result sets BEFORE running the potentially expensive filter on the invoice reference field. Again, assuming the invoice reference filter is expensive, this can massively improve the performance of our query. 

## Finally
Hopefully this helps summarise what OPTION FORCE ORDER does, and how it can be of use. Tuning sql queries is a bit of an art form, so if you do ever decide to add a hint to a query, make sure you understand its effects on your query first.

