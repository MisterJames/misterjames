title: Five Days of Azure Functions
layout: post
tags:
  - Azure Functions
  - c#
  - 5 days of Azure Functions
categories:
  - Development
authorId: james_chambers
featureImage: https://jcblogimages.blob.core.windows.net:443/img/2016/11/the-functions-are-coming.png
originalUrl: 'http://jameschambers.com/2016/11/Five-Days-of-Azure-Functions/'
date: 2016-11-11 15:03:50
---

I'm exploring the how-to's in Azure Functions that go beyond hello world.  I'm going to take the next five days to cover some interesting scenarios that I know I will find useful in my upcoming projects and hope that you can find value in it, too.  

![Azure Functions](https://jcblogimages.blob.core.windows.net:443/img/2016/11/the-functions-are-coming.png)

<!-- more -->

## Here's the topics
{% include_markdown _includes/Five-Days-Menu %}

## What you need to know
**Azure Functions are built on the Web Jobs SDK** which is a proven base that has matured over the last few years. It differs in that you can opt to use a "dynamic" pricing model rather than the "App Service" model. This is important, as you can now be billed per "gigabyte second", a new, ridiculously small unit of measure that clocks down to the milisecond. 

**c# support is provided through .csx files** which helps eliminate some of the cruft of projects, but introduces other limitations. Things like dependency injection aren't supported yet, and there is a little more legwork in getting third-party binaries up and available in your 

**Some libraries are preloaded to make things easy** and others are hot in Azure so you can reference them without having to pull in libraries manually. 

Here are the namespaces that are included in all your scripts by default. These namespaces are imported by default and are available as though you've already put the namespaces in `using` statements: 
```
Microsoft.Azure.WebJobs
Microsoft.Azure.WebJobs.Host
System
System.Collections.Generic
System.IO
System.Linq
System.Net.Http
System.Threading.Tasks
```

These .NET Framework assemblies are also available, but you'll have to add a `using` for any types you wish to use in your functions.
```
mscorlib
Microsoft.Azure.WebJobs.Extensions
System.Core
System.Net.Http.Formatting
System.Web.Http
System.Xml
```

There are other assemblies that are "hot" in the environment and can easily be brought into your scripts. If you want to take a dependency on a types in these libraries you need to reference them in your script:
```
Microsoft.AspNEt.WebHooks.Common
Microsoft.AspNet.WebHooks.Receivers
Microsoft.Azure.NotificationHubs
Microsoft.ServiceBus
Microsoft.WindowsAzure.Storage
Newtonsoft.Json
```

To create the reference to a library in your scripts, say for `Newtonsoft.Json`, use the following statement at the top of your script:
```
#r "Newtonsoft.Json"
```

Then you can add an appropriate `using` statement and use it's types.

**ASP.NET Core is not yet supported in Azure Functions** but support is on the way. This is a priority for the team and they are working hard on getting ASP.NET Core support, but there are still dependencies on too many libraries that are not yet ported to Core, as evidenced by the automatically "known" libraries that are included in Functions.  

## For more of the basics...
You can find more of the basics covered on the Azure Functions [documentation website](https://azure.microsoft.com/en-us/documentation/articles/functions-overview/), but if you're comfortable with the above, feel free to browse the articles in this series for some real-world ways to leverage Azure Functions.

## Here's the topics
{% include_markdown _includes/Five-Days-Menu %}

Would you like to see more? Suggest an Azure Function topic in the comments below or ping me on the Twitters.

## And Check Out the Monsters...
We got to mash with Chris Anderson who works on the Functions and Web Jobs team at Microsoft.

<iframe src="https://channel9.msdn.com/Series/aspnetmonsters/ASPNET-Monsters-78-Azure-Functions-with-Chris-Anderson/player" width="640" height="360" allowFullScreen frameBorder="0"></iframe>

Happy Coding!



