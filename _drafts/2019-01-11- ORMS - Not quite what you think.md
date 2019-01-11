---
layout: post
title:  The curious case of ORM Button (yes thats Object Relational Mapper)
tags: c# dotnet oo object-oriented development software microsoft ORM entity-framework linqtosql
---

## Intro
I'm writing this as a sort of reflection of my time using, or trying to use ORM's in .Net over the years. It's been an interesting journey starting with LinqToSql to EF to SQL, and I've come to a few conclusions over this time. When I first started wanting to use ORM's (years ago), the senior developers around me were deeply sceptical and I was confused as I thought ORM's were the solution to all of my database problems. As time went by, I discovered the reason for the scepticism and I wanted to record what I've found, and hopefully it may help guide someone else in the right direction (and maybe give them a laugh). 

So our question
 > Should I use an ORM?

This question is more complicated than it first sounds, theres quite a lot to think about with it. First we need to set the scene a little.

## The types of ORM
There are two main 'types' of ORM available:

### Full Featured ORMs
These tend to be a full featured database, but translated to Objects inside some sort of context object. They handle things like foreign keys, and quite often use Linq in C#, which is translated to sql when executing queries against it. Some examples of this type of ORM would be EntityFramework, LinqToSql and NHibernate (which I haven't tried before).

### Micro ORMS
These are much, much lighter weight and generally just allow you to execute SQL queries and automatically map the results to objects. They won't do relationships usually, and they will mostly focus on single isolated queries at a time.

## The Problems with Full ORMs
### They generate some terrible sql
### They don't perform well
### They don't scale particularly well
### Schema management is difficult to do in a team

## The Good bits of Full ORMs
### They are easy to use
### For 


## The Problems with Micro ORMs
## The Good bits of Micro ORMs
