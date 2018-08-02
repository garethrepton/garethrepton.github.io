---
layout: post
title: A tool to keep an eye on the big stuff... and the little stuff (NDepend)
tags: c# dotnet oo object-oriented development software microsoft
---

## Intro
I got a message from the guys at NDepend asking me to try out a full version of their software, and I'm a big proponent of using tools if they assist the development process, and I've seen NDepend around over the years, but never had the time to really make a case for it, so I thought why not. 

And I'm glad I did... 

### So what is it and what does it do?
NDepend is a little different to your standard dev tools, most tools are there to assist with the development process, I see NDepend as a tool to assist in the monitoring of a project. It does things like: 

 - Provides a report of the various stats of a project (i.e. lines of code, number of types, size etc...).
 - Monitors the project for various low level rules.. i.e. Source file organisation -> more than one type per file, naming conventions etc...
 - Monitors the project for high level metrics, i.e. Dependency graph violations, poor cohesion in assemblies etc...
 - Provides a technical debt report for the project.
 - Produces dependency graphs and analysis for a project.
 - Monitors all of the above over a series of time.

In my view, this marks NDepend down as a tool that is most useful for architects/senior developers who need to monitor the code quality of a DotNet project, but also need to get on with their own work too. 

----------------
##The stats
The vast array of project statistics available in NDepend really are great, and they offer a real insight into a solution that you really can't get anywhere else. 

##The charts

##The timeline feature
This, is really where NDepend comes into its own. Consider a situation where you've just been through a major release, accruing the odd bit of technical debt along the way. Most teams won't have the time to sit down and just clean all this technical debt in one sitting. What you really want to be able to do is implement some coding standards/ ideals and check that these are being followed and monitor the technical debt is decreasing in a project as you go. 

This is what the reports offer in NDepend. Each analysis is saved, and if you go into the project properties, you can select a baseline report to use, and just keep producing reports for a project for a number of months. And then look at the trend
