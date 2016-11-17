title: How to organize types in your Azure Function scripts
layout: post
tags:
  - Azure Functions
  - c#
  - 5 days of Azure Functions
categories:
  - Development
authorId: james_chambers
featureImage: https://jcblogimages.blob.core.windows.net:443/img/2016/11/types-in-functions.png
originalUrl: 'http://jameschambers.com/2016/11/How-to-organize-types-in-your-scripts/'
date: 2016-11-14 7:00:00
---
Even when you're dealing with Function Apps that have limited scope it's a good idea to break your scripts up into manageable, possibly reusable chunks. This is especially true if you want to work with the same data in several Functions in your app.  

![Keep your functions type-free](https://jcblogimages.blob.core.windows.net:443/img/2016/11/types-in-functions.png)

This is one way you can organize your scripts, types and objects in Azure Function Apps, and we'll have a deeper look at another approach later in this series.

<!-- more -->

{% include_markdown _includes/Five-Days-Overview %}

## A bit of a primer
The automatic bindings in Azure Functions are pretty nifty and cut out a lot of the communication and serialization cruft you might otherwise have to deal with.  You'll see function signatures like the following:

```
public static void Run(Stream imageBlob, string name, TraceWriter log) { ... }
```

Above, `Stream` is not a value type, it's a reference type...a complex object that is hydrated by the runtime for you. These parameters are bound for you when the function is executed and values from the input - be it a new file created on a file commit in blob storage, an HTTP request or some other trigger - will be mapped into the types you provide in your method signature. You can think of this as model binding as we know it in ASP.NET MVC (if you're from that background).

There is a step in the creation of your Function where you create the mappning for these bindings, and different types of Functions seem to have different capabilities for binding. For example, your input bindings for Queues seems to be more powerful (can bind to POCOs) whereas the HTTP-triggered Functions seem to only allow for the HTTPReqeust binding, meaning you'll have to deserialize the payload yourself.

## Create separate files for your types
The first thing you'll want to do is to _not_ put your types in your scripts, unless it's truly a single-purpose Function. In the code editor you can reveal your project assets by clicking the View Files button.

![](https://jcblogimages.blob.core.windows.net:443/img/2016/11/view_files_functions.png)

In the bottom of that tool pane, click the add button and create a new file. Here, my `person.csx` script has a class definition in it. 

```
public class Person {
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```

## Include your types in your Functions
When you define types in other files you will need to pull them into your Function, they are not inherantly available. You can pull in a type in another script as you would pull in a reference to an assembly:

```
#load "person.csx"
```

This allows you use the type either as a binding parameter (if supported for your Function trigger type) or as an instance you create in code. Here's an of using the `Person` class in an HTTP-triggered Function:

```
#load "person.csx"

using System.Net;
using Microsoft.Azure.WebJobs.Host;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
{
    log.Info($"Processed: RequestUri={req.RequestUri}");

    var person = await req.Content.ReadAsAsync<Person>();

    if (person == null)
        return req.CreateResponse(HttpStatusCode.BadRequest, "Person represented as JSON not found in request body.");    

    var name = $"{person.FirstName} {person.LastName}";
    log.Info($"Name: {name}");

    return req.CreateResponse(HttpStatusCode.OK, $"Hey there, {name}!");
}
```

Now, should you pass in a payload on an HTTP request with a JSON body of something like the following, your Function will be able to read that data out with `req.Content.ReadAsAsync<Person>()`:

```
{
	"FirstName" : "James", 
	"LastName" : "Chambers"
}
```

## Include using statements for default imports
Even though there is a default set of namespaces available to you in your Function scripts, I think it is advisable to explicitly declare your `using` statements. This will prevent problems with namespace conflicts, make it more apparent to others where your types are coming from (including future you, who tends to disapprove of older-you's shortcuts), and make it easier if you want to move your types out of the cloud and into reusable libraries. 

## Next Steps
Here are some things to try out:
 - Create a type in its own file in the Function Apps portal
 - Reference the type in another script and use the type in your Function
 - Try invoking the Function with Postman or another tool that lets you send HTTP requests

## Other articles in this series
{% include_markdown _includes/Five-Days-Menu %}

Happy coding!  


