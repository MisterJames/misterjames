title: Fan out workloads in Azure Function Apps
layout: post
tags:
  - Azure Functions
  - 5 days of Azure Functions
categories:
  - Development
authorId: james_chambers
featureImage: https://jcblogimages.blob.core.windows.net:443/img/2016/11/chained-function-results.png
originalUrl: 'http://jameschambers.com/2016/11/Fan-out-workloads-in-Azure-Function-Apps/'
date: 2016-11-16 06:51:32
---
You want to scale wide. That is it. That is the trick and the magic and the secret sauce of cloud computing. Do as much as you want to, as long as it's as little as possible. Break your work down into smaller parts and let Azure Functions handle the scaling part.

!["Nothing" scales really well. - Scott Hanselman](https://jcblogimages.blob.core.windows.net:443/img/2016/11/chained-function-results.png)

<!-- more -->

{% include_markdown _includes/Five-Days-Overview %}

Worksloads can come in many shapes and your goal as a developer in the cloud is to make sure that everything you touch happens quickly. API surface area that is required to scale has certain responsibilities such as keeping a low overhead, not doing IO bound operations synchronously and more. Azure Function Apps can help you achieve these goals by giving you building blocks that let you fan out your workload and keep your services nice and responsive.

**TL;DR:** Queue the work to be performed individually, then acknowledge the receipt of work. Let the actual processing happen in the background.

## An Example Scenario
Let's suppose you want to accept a list of photo albums to download. There could be dozens of albums and each album could have dozens or even hundreds of photos. You don't have the details of each album, so when you get the list, you'll need to iterate over all the albums and fetch the details, at which point you'll have the list of photos you can download.   

I'm using this example because it is close to my career: I actually built something like this many moons ago, prior to cloud being a "real" thing that we could tap into. The solution back then was to have many services running on many servers and using a single dispatcher to queue up downloads and distribute the download tasks. It cost many dollars to service the lease on those machines. Today, building something like this on Azure Functions is way easier, and wouldn't cost anything (infrastructure-wise) until users were paying to use it.

## Fanning Out for Scalability
To follow along with this one, you'll want to hit the Azure portal and create a Queue-triggered function with a C# template, creating the appropriate queue and selecting a storage account. Add a file to the function called `Album.csx` with the following code inside it:

```csharp
public class Album 
{
    public string Title {get; set;}
    public string URL {get;set;}
}
```

Our function will be receiving a message that a number of albums will need to be downloaded. The Functions UI has a handy "test" feature that you can use to send messages while you mash on your code. I'm using the following test data to simulate the information that would be coming into my queue:

```javascript
[
	{
		"Title" : "Last Summer", 
		"URL" : "http://awesomephotoshareR.com/albums/last-summer"
	},
	{
		"Title" : "Spring Break with the Kids", 
		"URL" : "http://awesomephotoshareR.com/albums/spring-break"
	},
	{
		"Title" : "Family Holidays", 
		"URL" : "http://awesomephotoshareR.com/albums/family-holidays"
	}
]
```

Here's my code to process the message:

```csharp
#load "album.csx"
using System;

public static void Run(List<Album> albums, ICollector<Album> queuedAlbums, TraceWriter log)
{
    foreach(var album in albums)
    {
        queuedAlbums.Add(album);
        log.Info($"Added {album.Title} to the queue.");
    }
}
```

Note that I'm just logging things out and returning. We need to make that acutally do some interesting work.  More importantly though, I'm not going to be doing any of the work here, not the downloading bits, anyways. I want simply to put the album information on the queue to be processed external to this call. I can do this quickly and get out.

You'll notice the `ICollector<Album>` in the signature. That is augmented from the default `ICollector<string>` so that we can take advantage of the type binding that Functions offers.

## Building a Function Chain
The next thing to do is to start sending that output _somewhere_. It's going to the queue, but it's not being processed.  To get processing to happen, we're going back to the integration tab and pressing the magic button:

![](https://jcblogimages.blob.core.windows.net:443/img/2016/11/function-chaining.png)

Clicking "go" takes you to the create page for adding a new function to your Functions App. The queue trigger template is selected and the parameters are already filled out. When you create the new function you're taken to the code, where the initial pass of the script looks like so:

```csharp
using System;

public static void Run(string myQueueItem, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
}
```

Let's update that, because we have a type already broken out, and we want to have more strongly-typed arguments.  Add an `Album.csx` to your function again here (it's a different file set from our first one) and put the same members in the class as before. Don't worry - we're going to look at fixing this copy-and-paste nonsense later in the series.

Your new function should look like so:
```csharp
#load "Album.csx"

using System;

public static void Run(Album album, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {album.Title}");
    // do the downloading bits here...    
}
```

There, now, if you want to see the fun stuff happen, open another browser window and put your two functions side-by-each on the screen. Return to the intake function and run it again. Your single-message list of albums is diced up and sent to the queue one-by-each, then processed over in the other function.  Cool beans.
 
![Jazz Hands!](https://jcblogimages.blob.core.windows.net/img/2016/11/chained-functions-output.png)

## Building Messages That Support Scaling
When you have the data at hand, don't squirrel it away from the rest of your processing pipeline. Instead, forward that information along.

We should always work towards having a measure of idempotence in our messages. What I mean by that is simply that the message should stand on it's own. If you have the title and the URL of an album to download, don't make the next handler in the chain use the URL to look up the title. It can mean that you start to build out fatter messages, but the payoff is that you don't have to go and look things up. Messages can be replayed without pulling in dependencies and you'll reach a much higher level of scalability. 

Note that it's a good practice in cases like this to include something along the lines of a correlation ID, to help understand which queued work items belong together. It's also a good way to figure out when you've completed a distributed set of work. If folks are interested in this, I will dig further into how to achieve this in Azure.

## Other articles in this series
{% include_markdown _includes/Five-Days-Menu %}

Happy coding!  
