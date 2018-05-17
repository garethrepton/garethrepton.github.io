---
layout: post
title: Reduce unit test friction patterns - 3 - Make your tests idempotent
tags: unittesting c# patterns autofixture
---

## The Problem
There are several possibilities for this, but lets concentrate on a bad one. You have a target class that spawns a thread which updates a field with its result and you want to test that the result is set correctly.

The really naive solution t