---
layout: post
title: TypeScript common linq command equivalents / CheatSheet
tags: typescript c# angular dotnet javascript cheatsheet
---

# Intro
Linq in c# is a great abstraction, it massively reduces the amount of code to do fairly basic operations, using TypeScript doesn't mean you lose this functionality, its just a little different to achieve many of the operations. In this article I'll run through the basic Linq-To-Object operations, and how you can achieve similar results in TypeScript.

For these examples, I'll keep it simple with a list of Person objects that look like this:

     {% highlight CSharp %}
      public class Person{
       public string Name {get;set;}
       public string Title {get;set;}
   }
     {% endhighlight %}

    {% highlight TypeScript %}
   class Person
{
    constructor(public Name: string, public Title: string)
    { }
}
     {% endhighlight %}

(How good is that typescript constructor/field syntax!).

With the data:
 - Chandler - Mr
 - Monica - Mrs
 - Rachel - Miss
 - Joey - Mr
 - Ross - Dr

----------------
# Select
To select the names of each person with their title:

### CSharp
{% highlight CSharp %}
people.Select(x => new { FullTitle = $"{x.Title} {x.Name}"});
{% endhighlight %}

### Typescript
In TypeScript, the equivalent to select is the map function:
{% highlight TypeScript %}
people.map(x => ({ FullTitle: x.Title + ' ' + x.Name }));
{% endhighlight %}

----------------
# Where
To filter the list to only the people with the title "Mr":

### CSharp
{% highlight CSharp %}
people.Where(x => x.Title == "Mr");
{% endhighlight %}

### Typescript
{% highlight TypeScript %}
people.filter(x => x.Title == "Mr");
{% endhighlight %}

----------------
# OrderBy
To order the list by Name:

### CSharp
{% highlight CSharp %}
people.OrderBy(x => x.Name);
{% endhighlight %}

### Typescript
In TypeScript, the equivalent to order by is the sort function, but you do have to give it a sort function that returns 1 or 0:
{% highlight TypeScript %}
people.sort((x,y) => x.Name > y.Name ? 1 : -1);
{% endhighlight %}

----------------
# GroupBy
To group the list into the various titles:

### CSharp
{% highlight CSharp %}
people.GroupBy(x => x.Title);
{% endhighlight %}

### Typescript
There isn't a simple equivalent to this, the closest thing is probably the reduce function:

{% highlight TypeScript %}
var grouped = people.reduce((g : any, person : Person) => {
    g[person.Title] = g[person.Title] || []; //Check the value exists, if not assign a new array
    g[person.Title].push(person); //Push the new value to the array
    return g; //Very important! you need to return the value of g or it will become undefined on the next pass
}, {});
{% endhighlight %}

----------------
# FirstOrDefault
To select the first person with the title Mr, or null if none exist:

### CSharp
{% highlight CSharp %}
people.FirstOrDefault(x => x.Title == "Mr");
{% endhighlight %}

### Typescript
{% highlight TypeScript %}
people.find(x => x.Title == "Mr");
{% endhighlight %}

----------------
# Aggregate
Lets concatenate all the names together:

### CSharp
{% highlight CSharp %}
people.Select(x => x.Name).Aggregate((x,y) => x = x + "" + y).Dump();
{% endhighlight %}

### Typescript
This time, reduce is almost like for like with the c# equivalent
{% highlight TypeScript %}
var concat = people.map(x => x.Name).reduce((g : any, name: string) => {
    g += name;
    return g;
}, "");
{% endhighlight %}


