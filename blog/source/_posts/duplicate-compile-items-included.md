title: Duplicate 'Compile' Items Included Error in Visual Studio
layout: post
tags:
  - ASP.NET Core
  - Visual Studio 2017
categories:
  - Development
authorId: james_chambers
featureImage: logo_579.png
originalUrl: 'http://jameschambers.com/'
date: 2017-11-12 16:00:00
---

This is an error I've gotten a few times lately in Visual Studio 2017 after renaming a file:

> Duplicate 'Compile' items were included. The .NET SDK includes 'Compile' items from your project directory by default. 

Thankfully, fixing this is pretty straightforward.

<!-- more -->

## How I Got Here

I'm not sure what series of events led you to getting this error message, but for my project it happened when I was renaming a file in Visual Studio 2017. In my case, I had a file that contained a couple of classes that I intended to break out into separate files. None of the classes assumed the name of the file. My file was unsaved and I attempted to rename the file in the IDE to the name of one of the classes.

The full error message is as follows:

> Duplicate 'Compile' items were included. The .NET SDK includes 'Compile' items from your project directory by default. You can either remove these items from your project file, or set the 'EnableDefaultCompileItems' property to 'false' if you want to explicitly include them in your project file.

After the IDE makes the change, I start getting the error message, along with all kinds of 'unambiguous reference' errors throughout my code.

The error message is offputting because it is worded in such a way that you've done something wrong. In my case, I haven't touched my project file, so what is it that is causing the project file to contain invalid sections in my config?  The rename process in Visual Studio must include some staging steps where it tries its best to keep track of the in-progress changes, but it apparently doesn't degrade gracefully.

Of course, if you've been meddling with the project file on your own, then you can't pin it on Visual Studio ;)

## The Fix

Reading the error message we can understand that the project file contains instructions to include a file explicitly in a 'Compile' section that is not required, because the file in question is already contained as part of the project path.

In the Solution Explorer, right-click on your project and select `Edit Project .csproj` from the context menu.

The full text of the error message will tell you the file name that is problematic, and it will be in a section that looks something like the following:

```
  <ItemGroup>
    <Compile Include="YourFolder\Namespace\ClassNameHere.cs" />
  </ItemGroup>
```

It is safe to remove this 'Compile' node, along with the wrapping 'ItemGroup', provided the file is indeed in your path.  Before nuking the node, create a commit in  your source control software so that you can roll back if need be.

Happy Coding!



