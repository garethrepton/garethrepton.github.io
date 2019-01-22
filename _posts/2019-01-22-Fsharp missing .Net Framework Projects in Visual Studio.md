---
layout: post
title: The solution to FSharp missing .Net Framework Projects in Visual Studio
tags: f# visualstudio c# dotnet projects
---

## Intro
So I went to create a .Net framework F# console app as part of a larger existing .Net Framework solution, and found that the project dialog didn't give me that option for a full .Net Framework project. I thought this was a bit weird, so did some googling and found that you have to install the **F# Desktop Language Support pack**.

## The solution
To do this, in the main Visual Studio menu go to:

 > Tools > Get Tools and Features.

On the installer dialog that appears select the "Individual Components" tab and find F# desktop language support:

![installer](/images/FSharpInstall/FSharpInstaller.png)

Then install that as a new feature.

Once installed, back in visual studio you should see:

![projects](/images/FSharpInstall/FSharpProjects.png)

Console, and Library will allow the selection of a standard .Net Framework project. This is ideal for integrating into existing code.










