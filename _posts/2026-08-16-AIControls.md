---
layout: post
title: If you don't check it, how do you know?
tags: #AI #RandomNotes 
published: false
---

*I'm starting to write some of my thoughts down as I start to join the dots a bit with AI partly as a means to hold myself accountable and increase my own knowledge, they will likely change frequently because its very new and the industry as a whole is still creating new approaches seemingly on a weekly basis.*

All the hype around AI is around how quickly it can make things, and it is very good at relatively shallow problems, and with the right guidance (quite a lot) can be good at more complex problems. But its probably the wrong thing to focus on, the harder and more important problem is how do you know what its created is correct and good enough for your purposes. 

Anyone can now make something with AI, look at the result, say that's about right, then just use that for every problem they ever have going forwards. But the likelihood is, it actually isn't correct and its laced with subtle bugs, or worse it could be doing something they don't think it is. Particularly if there is a non trivial amount of logic involved, or a significant amount of bespoke context.

Its a slightly ridiculous example, but can you imagine a calculator app where only a few cases have been tested and asked for so the LLM wrote it as a switch statement? If the consumer is non-technical, they wouldn't know, they'd just find out the first time they tried to use it for something other than their original test case. Now imagine that sort of thing baked into an application somewhere churning out slightly incorrect numbers over and over because it was never checked.   

Even getting an independent model to verify the work of another model via agents seems fraught with issues as there seems to be a significant overlap between frontier models even from different providers as they share a level of training data, and are all ultimately very similar in how they actually work. This means they agree on the same mistakes!

Right now it doesn't seem possible to actually validate AI generated solutions without an experienced engineer reviewing what its created, providing shedloads of context, guiding automated tests (which AI greatly helps with), and some levels of manual testing, and even then it can be hard to gain confidence. Maybe I am missing something here, if I am I would love to know!

What AI actually creates is this slightly odd blend of non deterministic meta programming, where the goal is no longer to write the actual code, but to create enough context, tests, reviews and guardrails to keep it on track for what you actually want to achieve. I find this way of working quite fun, and has a low barrier to entry, but it feels like the skill ceiling could be quite high as the actual best engineers using it will be able to back what they have produced with its obvious productivity gains, not just produce untested working prototypes really quickly.

#### As a side note

I wonder how many LLMs checking each other you'd have to chain together before you could call work produced by AI effectively as deterministic as something created by a person without ever looking at it.