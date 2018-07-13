title: 'Removing Bower - The command "bower install" exited with code 1 When Publishing'
layout: post
tags:
  - ASP.NET Core
  - ASP.NET Core MVC
  - Visual Studio 2017  
  - npm
  - bower
categories:
  - Development
authorId: james_chambers
featureImage: logo_579.png
originalUrl: 'http://jameschambers.com/'
date: 2018-07-12 5:42:00
---

I was cleaning up and updating the JS package management in a project that has shuffled along from ASP.NET Core 1.0 (and runs on .NET 4.6) and everything seemed to be going fine. And then I tried to publish. 

> Publish has encountered an error. Build failed. Check the Output window for more details.

Upon further examination, I noticed that there was a problem with an msbuild task, namely `PrepublishScript`.

<!-- more -->

The error looked something like the following:

```
    Target "PrepublishScript" in project "C:\james\code\project.csproj" (target "PrepareForPublish" depends on it):
        Using "Exec" task from assembly "Microsoft.Build.Tasks.Core, Version=15.1.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a".
        Task "Exec"
        Task Parameter:Command=bower install
        Exec: bower install
        Exec: bower                           ENOENT No bower.json present
C:\james\code\project.csproj(98,5): Error MSB3073: The command "bower install" exited with code 1.
        Done executing task "Exec" -- FAILED.
        Done building target "PrepublishScript" in project "myproject.csproj" -- FAILED.
```

This made sense; after all, I had just deleted my `bower.json` and moved package management over to npm. 

To clean this up, I had to simply unload the project and edit the csproj file directly. Tracking down the `PrepublishScript` target, I was able to locate the line that was causing me grief:

```
  <Target Name="PrepublishScript" BeforeTargets="PrepareForPublish">
    <Exec Command="bower install" />
    <Exec Command="dotnet bundle" />
  </Target>
```

Removing the `exec` line for bower did the trick, and I was able to resume publishing.  

As a final step, I had to reload my project and reset my startup project configuration, which Visual Studio loses when you unload a project in the mix.

## Why This Happens
Over time the templates for projects evolve and they may or may not leave artifcats behind, such as a build step for a now antiquated package manager.  You may see other errors such as this in your travels, especially on projects created "in transition" as ASP.NET Core came about.  Don't be afraid to create a branch and play with that csproj file if needed when you run into these kinds of things; you can always roll back your changes or abandon the branch if things go south.

Happy coding!
