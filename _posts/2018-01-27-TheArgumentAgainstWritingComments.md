---
layout: post
title: The argument against writing comments
---

tldr; Comments imply the code does not speak for itself.

Code with a lot of comments, usually has the comments because the code itself doesn't adequately represent what its doing, and in most cases it could be changed so that it does describe itself correctly without a huge amount of upfront effort (apart from a few oddities to do with technological limitations, or odd requirements imposed on the code base).

Like documentation (which is fine as a communication mechanism btw.) comments are often out of date just after they are written. Developers do not update them when bug fixing, and often when changing a feature completely as they are not part of the code, there is nothing compiling them and telling them they are completely incorrect. This means many of the comments in a code base are often out of date and completely incorrect. 

{% highlight c# %}
//Add A to B
var c = A - B;
{% endhighlight %}


I'm not saying comments are all bad, a comment is perfectly reasonable where perhaps you know something could be written better, but you don't have enough time to possibly do it and want to tell developers this, or you run into some odd limitation in the technologies you are using and have to put a hack in just to make them work in a reasonable way (happens quite a lot). But I am saying its far far better for the code to explain itself, and you end up with a better design because of it. Also see: http://butunclebob.com/ArticleS.TimOttinger.ApologizeIncode and countless other articles on the subject.

Furthermore, Object Oriented languages are about encapsulating behaviour and data into small manageable chunks that each do their own thing (ideally just one thing), and when combined form a greater whole to form an application. This allows us to break down and more easily visualise the inner workings of a program so that we can see and understand what is going on, which can then be translated into machine code that the computer understands. This is fundamentally the point of object oriented languages, to take something complex, and break it down into small manageable chunks so we can focus on and TEST just one thing at a time, whilst still working towards the larger application. It follows that if everything is an individual component that just has one purpose, its definition should be clear and concise and not require further explanation.

Just to clarify, I'm not saying never write comments here, I'm saying if you can express the meaning of a piece of code within the code itself, do this, it bypasses many of the issues you might have with comments, and makes the code more readable.

If, for example, external library A requires you to call some a bizarre series of methods in some obscure order, yes... write a comment, for everyones sanity.

