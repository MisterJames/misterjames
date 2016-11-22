title: Writing Azure Functions in F#
layout: post
date: 2016-11-16 17:00:00
categories:
  - Development
tags:
  - Azure
  - Azure Functions
authorId: simon_timms
---

*Thanks* Thanks to James for letting me guest author on his blog. I love funcitons and I love F# so this was a fun post to write. 
-Simon

Azure functions can be written directly in a wide variety of languages

* Bash
* Batch
* C# 
* F#
* Python
* JavaScript 
* Powershell

You can also build an executable which can run on Windows and upload it to the function to be executed. This approach allows for maximum compatibility with most any workload you can throw at it. Of all the natively supported languages I think F# fits most nicely into the functional mold. 

![azure_functions_fsharp_title.png](/content/azure_functions_fsharp_title.png)

<!-- more -->

Of course you can write your functions in C# and everything will be fine but wouldn't you rather write them in a functional programming language? 

F# isn't perhaps a pure functional programming language but it is close enough to have some great advantages over C#. Declarations are immutable by default, the syntax is terse and stylish, null exceptions are all but unheard of and it has fantastic support for filter which are useful when dealing with any sort of complex data. It is a natively supported language on Azure Functions so let's see how that works. 

You can start with a new Azure function but instead of selecting C# let's take F#. For our example we'll base it off of the HTTP Triggered function.

![azure_functions_fsharp_select_trigger.png](/content/azure_functions_fsharp_select_http_trigger.png)

In our scenario we'd like to pull back the headlines from the BBC World Service in JSON format. This, as it turns out, is pretty easy to do in F# thanks to type providers. 

A type provider is a compile-time shim which can be used to generate types from a variety of different data sources. The FSharp.Data project includes providers for such things as CSV, XML, Json and databases among other data sources. You can read more about it at [their website](https://fsharp.github.io/FSharp.Data/index.html). We can start our project by adding a new file to the solution, a project.json. This will allow downloading and including libraries from nuget. 

Our project needs the type provider and also a handy Json serializer. We'll include the latest FSharp.Data and Newtonsoft.Json:

```
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "FSharp.Data": "2.3.2",
        "Newtonsoft.Json": "9.0.1"
      }
    }
   }
}
```

Once you save that file you should see a restore started by the functions runtime. This will download and stage your libraries. 

Now onto the actual F#. Start by opening the run.fsx file anc clearing it out. We'll need to include the libraries we've just downloaded form nuget

```
#r "System.Net.Http"
#r "FSharp.Data"
#r "Newtonsoft.Json"

open System.Net
open System.Net.Http
open FSharp.Data
open Newtonsoft.Json

```

With all our libraries properly referenced we can pull the definition of the BBC's RSS feed into a type. Remember this will be done at compiles time so you don't have access to runtime variables like settings in the environment. 

```

type RSS = XmlProvider<"http://feeds.bbci.co.uk/news/rss.xml">

```
As simply as that we now have a type called RSS which is generated from the BBC's RSS feed. This could have been any feed or format, the XmlProvider just downloads the file and infers the schema.

Now the meat of the problem, download the feed, get the titles and convert them into Json

```
let Run(req: HttpRequestMessage, log: TraceWriter) =
    log.Info(sprintf 
        "F# HTTP trigger function processed a request. ")
    
    let articles = RSS.GetSample()
    log.Info(articles.Channel.Title)

    let titles = articles.Channel.Items |> Seq.map(fun x -> x.Title )
    req.CreateResponse(HttpStatusCode.OK, JsonConvert.SerializeObject(titles));
```
This code downloads the RSS feed, logs out the title then maps out the item titles and returns it as a Json serialized array. If you're new to F# you might have noticed that we don't explicitly return anything. F# will implicitly return the last line of your function. 

At the time of writing the output of this function looks like 

```
"[\"Hillary Clinton: I wanted to curl up with book after election loss\",\"Liz Truss urged to 'get a grip' on minimum-term inmates\",\"Whiplash plans to 'cut car insurance premiums by £40'\",\"Plans to curb House of Lords powers 'dropped'\",\"Leonard Cohen: Singer died in sleep after fall\",\"Price of Football 2016: Premier League cuts cost of tickets\",\"Dementia game 'shows lifelong navigational decline'\",\"Animals still poached in 'horrifying numbers' - Prince William\",\"Online calculator predicts IVF baby chances\",\"Hillsborough: Sir Norman Bettison defends book on disaster\",\"RSPB hails 'remarkable' recovery of threatened cirl bunting\",\"Most common surnames in Britain and Ireland revealed\",\"Meet the girl, 4, who called 999 and saved her mum's life\",\"Donald Trump's name removed from NYC buildings\",\"Price of Football 2016: Away tickets can cost more in Championship than Premier League\",\"Mosul battle: Inside an Islamic State mortar factory\",\"Airbus crew train in Stornoway crosswinds\",\"The video game that's actually dementia research\",\"China traffic dance video goes viral\",\"Part of an Eiffel Tower staircase up for auction\",\"BBC Breakfast\",\"What will President Trump do about North Korea?\",\"Phil Mercer: Australia's child poverty 'national shame'\",\"Is Nigel Farage heading for the Lords?\",\"Pidgin - West African lingua franca\",\"Trump presidency: Your questions answered\",\"How does the UK's Supreme Court work?\",\"Newspaper headlines: Steak in prison and 'Three Lions party'\",\"The Blood Forest\",\"Supermoon\",\"Picking up the pieces\",\"Week in pictures: 5 - 11 November 2016\",\"Your pictures\",\"Andy Murray beats Kei Nishikori at ATP World Tour Finals in London\",\"Wayne Rooney apologises to England chiefs over 'inappropriate' images\",\"Women's Champions League: Brondby 1-1 Manchester City Women (agg 1-2)\",\"Brackley Town 4-3 Gillingham (aet)\",\"Whites v blacks\",\"'He's a devil'\",\"Price of Football\",\"Sick and stranded\",\"Battle of the barnets\",\"Fake news quiz\",\"Housing squeeze\",\"Virtual Ariel\"]"
```

Of course the F# used here isn't very idiomatic. Let's clear it up a bit 

```
let Run(req: HttpRequestMessage, log: TraceWriter) =
    log.Info(sprintf 
        "F# HTTP trigger function processed a request. ")
    
    req.CreateResponse(HttpStatusCode.OK, 
        RSS.GetSample().Channel.Items |> 
        Seq.map(fun x -> x.Title ) |> 
        JsonConvert.SerializeObject
    )
```

With that we have an Azure Function written in a functional programming language. 20 lines all told, including blank lines.


# Other articles in this series

Day 1: [How to organize types in your Azure Function scripts](http://jameschambers.com/2016/11/How-to-organize-types-in-your-scripts)

Day 2: [How to resize an image uploaded to Azure Blog Storage](http://jameschambers.com/2016/11/Resizing-Images-Using-Azure-Functions)

Day 3: [How to "fan out" work so your Functions can scale](http://jameschambers.com/2016/11/Fan-out-workloads-in-Azure-Function-Apps)

Day 4: How to deploy to Azure Functions using GitHub (Up Next!)

Day 5: How to import third-party libraries (Coming soon)
