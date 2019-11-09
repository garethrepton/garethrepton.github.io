---
layout: post
title: The end of the using block in C#?
tags: C# Sql TSql Query Tuning SqlServer visualstudio dotnet projects database query optimisation
---

## Intro
C# 8 unfortunately seems to be branching away from the standard .Net Framework and its not (fully?) compatible with the full .Net Framework, instead you have to use DotNetCore to get the benefits of it. 

That being said One of my favourite new features is the automatic using block scoping. Anything that removes some of the verbosity of the nested curly braces is good as far as I'm concerned.  

## Whats changed?
Its actually fairly simple, lets take a disposable class called "Test" as an example:

{% highlight CSharp %}

    public class Test : IDisposable
    {
        private string name;

        public string Name => name;

        public Test(string name)
        {
            this.name = name;
        }

        public void Dispose()
        {
            Console.WriteLine($"ending {name}");
        }
    }

{% endhighlight %}

Now, to recap, the traditional way to use this class in C# is to declare your using block, and then either a single line with no curly braces, or add curly braces for multi line usage, like this:

{% highlight CSharp %}

    static void Main(string[] args)
    {
        using (var t2 = new Test("t2"))
            DoSomethingWithTest(t2);

        using (var t3 = new Test("t3"))
        {
            DoSomethingWithTest(t3);
            DoSomethingWithTest(t3);
        }
    }

    static void DoSomethingWithTest(Test test)
    {
        //Example only
    }

{% endhighlight %}

This is fine, but... it does nest your code 1 level deeper, again this is fine in most cases, but combine this with a couple of if statements and the code becomes pretty messy.

When the end of the declared using block is reached the object is automatically disposed, so we get a printout to the console like this:

<div class="consolenosize">
  <div class="consolebody">
    <p class="consoletext">Done something with t2
                            ending t2
                            Done something with t3
                            Done something with t3
                            ending t3
    </p>
  </div>
</div>

### The new sound
You can still use the old way shown above in C# 8, but the new way of declaring usings, is to scope them to your method like this:

{% highlight CSharp %}

    using var t = new Test("t");
    using var t1 = new Test("t1");
    
{% endhighlight %}

This tells the compiler that you want your disposable object to live as long as the scope it is declared in (the method its in). So `t` and `t1` will automatically get disposed at the end of that method block. So... putting this back into our example we get the following code:

{% highlight CSharp %}

        static void Main(string[] args)
        {
            using var t = new Test("t");
            using var t1 = new Test("t1");
            
            using (var t2 = new Test("t2"))
                DoSomethingWithTest(t2);

            DoSomethingWithTest(t);

            using (var t3 = new Test("t3"))
            {
                DoSomethingWithTest(t3);
                DoSomethingWithTest(t3);
            }

            DoSomethingWithTest(t1);
        }

        static void DoSomethingWithTest(Test test)
        {
            Console.WriteLine($"Done something with {test.Name}");
        }
    
{% endhighlight %}

This prints:

<div class="consolenosize">
  <div class="consolebody">
    <p class="consoletext">
        Done something with t2
        ending t2
        Done something with t
        Done something with t3
        Done something with t3
        ending t3
        Done something with t1
        ending t1
        ending t
    </p>
  </div>
</div>

So as you can see, our t2 and t3 instances end much sooner than the t1, and t instances. But otherwise, we can use the variables in much the same way as with traditional usings, we just don't have to worry about where they get disposed. 

## Finally
Personally I think this leads to much cleaner code, hopefully you do too.

