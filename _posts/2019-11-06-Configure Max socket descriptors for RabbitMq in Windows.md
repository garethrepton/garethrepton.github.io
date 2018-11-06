---
layout: post
title: Configure max socket descriptors for Rabbitmq in Windows
tags: c# dotnet oo object-oriented rabbitmq messagequeues
---

## How to..
This is quite frankly a bit weird so I thought I'd document it....

You don't set the max number of socket descriptors in RabbitMq via the rabbit configuration, its set via the Erlang environment variable on the local machine, via an admin command prompt you can do it like this:

{% highlight bash %}
setx ERL_MAX_PORTS <Your max number goes here> /m
{% endhighlight %}

and thats all there is to it, just be careful not to set it to the maximum available on the operating system you are running.
