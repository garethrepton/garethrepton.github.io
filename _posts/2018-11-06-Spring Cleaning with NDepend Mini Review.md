---
layout: post
title: Spring cleaning a visual studio solution with NDepend & Mini Review
tags: c# dotnet oo object-oriented development software microsoft
---

## Intro
I got a message from the guys at [NDepend](https://www.ndepend.com/) asking me to try out a full version of their software, and I do like a good tool to simplify some of the more complicated development tasks, so heres a really quick post on what I found most useful on initial usage.

### So what is it and what does it do?
NDepend is a little different to your standard dev tools, most tools are there to assist with the development process, I see NDepend as a tool to assist in the cleaning up and monitoring of a project to prevent it becoming unruly. It does things like: 

 - Provides a report of the various stats of a project (i.e. lines of code, number of types, size etc...).
 - Monitors the project for various low level rules.. i.e. Source file organisation -> more than one type per file, naming conventions etc...
 - Monitors the project for high level metrics, i.e. Dependency graph violations, poor cohesion in assemblies etc...
 - Provides a technical debt report for the project.
 - Produces dependency graphs and analysis for a project.
 - Monitors all of the above over a series of time.
 - Diff all of the above since a user defined baseline.
 
Thats a lot of stats and tools... I'm going to look at a very small subset of this for the purposes of this post, because theres just so much in there, and to be honest, this is probably the first thing people are likely to do when getting into using the tool. I'm going to focus on things that help clean up/monitor an existing side project visual studio solution I have lying around.

## The Rules engine
NDepend has a bunch of useful rules in it, and once it has analysed a solution, gives you a summary of the rules which are violated. On a small project, these are normally easy to spot manually, but on a large project, this view gives a really useful summary of the state of a project. Although it should be stated, it does need interpretation, as usual in coding the answer is often "it depends" and not every rule violation is necessarily something that needs cleaning up, but this is to be expected.

Now, I'm not going to list all of the rules available, but here are some of my initial favourites that tend to be particularly difficult to spot in a visual studio solution, but NDepend makes pretty easy to see and cleanup.

### Avoid Defining multiple types in source code files
In a world where the folder structure of a solution is far less important with ReSharper installed (thanks for Ctrl+T), as soon as I realised that this rule existed, I knew I should check my sample solution with it. Turns out I created a lot of classes in the same files... easy to do, but this rule is pretty handy for keeping on top of it. Especially in a solution where multiple developers were committing to the project. 

### Namespace name should correspond to file location
This one speaks for itself, its quite handy just to see all of the files where their folder does not match their namespace. Seeing them in one place allows you to pick and choose the worst offenders, and monitor for issues getting introduced. Combine this with the ReSharper refactor feature of selecting a folder and moving types into matching namespaces, this makes a very fast way of cleaning up these sorts of issues in a solution.

### Avoid having different types with the same name
This one is probably more applicable to larger projects, but very useful. It will tell you if there is anything in your solution in different namespaces, but with the same name. This can be pretty annoying when it happens as its easy to include the wrong one in your code and end up with weird behaviour.

## Other useful metrics/tools
I found that there were some other immediately useful and intuitive features that also helped when cleaning the project up:

### Count of Lines of code
This is quite an accurate reflection of the number of lines of code in the solution. Although this isn't necessarily very useful in isolation, whilst you are cleaning up a solution it can be quite a good indicator of how its going. I found this particularly useful for seeing just how much unused code I had deleted.

### Ability to save a snapshot of your code & compare to previous baseline snapshots
This is a really good feature, it allows you to store a snapshot, make a bunch of fixes to a codebase, and then compare that snapshot to the new snapshot and essentially see your progress on the charts provided. Very useful for keeping things on track over time, and also quite motivating whilst cleaning things up.

### Dependency Graphing
This is probably the best auto generated dependency graph tool I've seen and it makes it fairly easy to see if inappropriate references have been added between solution projects. 

## Wish list
I've barely scratched the surface of the features available, but I've thought of a couple of features I think would be great:

### Async/Await rule set
One of my bug bears in c# is the ease at which a developer can cause chaos (bring down the main thread for example) by not awaiting an async method call. It would be great to see a set of rules in NDepend that highlight potential violations of this. Things like not awaiting a particular method result, or returning void from an async method would truly help, and fit quite neatly into the set of rules that are in NDepend.

### Dependency mismatch issues
It can be quite easy in a solution to end up with multiple dependencies within different projects that are potentially incompatible, and/or not found, especially when certain Nuget packages like to redirect in the app.config file. For example if you have a version of Newtonsoft.Json thats 4.3 everywhere, but then you install a separate Nuget package into your solution that requires v4.6, and that redirects the dependencies from the app.config file to 4.6 rather than 4.3. This can subsequently cause build & runtime errors on a CI server because the project will be looking for 4.6 when in reality it should be looking for 4.3. Perhaps it would be possible for an NDepend to detect and highlight these instances too, although this might be a big ask, it can be a real pain to debug though.

## In Summary
This has been a really quick roundup of some of the features I found immediately useful with NDepend whilst applying it to an old side project solution I had lying around. I've barely scratched the surface of its features, but I did find it pretty useful. Whilst I don't think it should be used without interpretation, if its the type of thing you need to monitor/cleanup in a project or solution, I can see how it could be a real time saver. It was certainly very useful for cleaning up the existing problems in this solution.


