---
layout: post
title: The TDD toolkit
tags: tdd unit-testing c# dotnet toolkit mocking 
---

I'm going to try to do a few posts on test driven development and unit testing in the .Net world. Yeah its been covered quite heavily already, but I think more information out there can only be a good thing, can't it?

### Testing Framework
There are a few very similar choices here, I quite like [XUnit](https://xunit.github.io/) - its data driven Theory style tests really quite useful, and its nice and simple to work with, although I'm sure [NUnit](http://nunit.org/) would provide nearly the same experience. Even MSTest wasn't too bad last time I used it.

### Mocking Framework
A good mocking framework is essential for unit testing in the .Net world, although there are other types of test doubles, mocks are really convenient when they fit the system under test. [NSubstitute](http://nsubstitute.github.io/) gets my vote on this. I was a long time user of [moq](https://github.com/Moq/moq4/wiki/Quickstart), but NSubstitutes support for recursive mocking (i.e. Interface.SubInterface.SomeMethod().Returns(1)) means it saves a significant amount of time and code over moq.

### Test Data Builder
The mighty [AutoFixture](https://github.com/AutoFixture/AutoFixture) is really powerful for this. In summary, it allows you to do clever things, like abstract away your target objects constructor from the tests, so you don't have to change all of your test methods whenever the constructor changes, it also allows you to automatically build most test data you may want to use during a test. Its probably best used sparingly until you learn how best to use it. 

### Syntactic Sugar
I'm a big fan of [Fluent Assertions](http://fluentassertions.com/) for my test assertions, I think it makes the tests more readable, and offers some handy helper shortcuts for things like exception testing.

### Test Runner
[NCrunch](http://www.ncrunch.net/) wins this hands down for me. It automatically builds and runs your tests in the background quickly and with minimal fuss, and highlights the code coverage against the system under test to give you an idea of areas you may have missed completely.  

### Continuous Integration Server & Source Control
Doesn't really matter which, but if you have a team of people working on something, set one up, and get it to automatically run your tests. 

And thats just about it, I'll add some more if I think of any, but these are the tools I've found to be most useful 


