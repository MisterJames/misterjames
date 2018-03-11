title: 'BakeBot - A Bot for our Bakery Built with the Microsoft Bot Framework'
layout: post
tags:
  - Bot Framework
categories:
  - Development
  - Chez Angela
authorId: james_chambers
originalUrl: 'http://jameschambers.com/2018/03/bots-day-1-What-a-bot-is/'
date: 2018-03-08 23:12:54
---


BakeBot Day 1: What a Bot Is (And What It Is Not)

One of the most important things in bot creation is understanding what a bot should actually be. The last thing we want to do is to over extend ourselves. Complexity is always the enemy when we're working on software, and bots have a very real human component in their routing operation that makes things complex enough on their own without us adding in any extra.

feature image

Before we dive into building a bot, let's start by getting on the same page of understanding and filling in some background details. If nothing else, this will give you visibility into why I've approached things the way I have.

<!-- more -->

## It's Like They Knew We Were Coming

Over the years I've made several stabs at building some type of command-and-answer service. Early on I wrote text games simliar to "Mad Libs".  I've written auto-responders for email clients and console applications that allow users to choose from menus. I have built context menus in WinForms applications, dropdowns on web pages and multi-view apps in mobile toolkits. I've written scripts that accept various commands, options and parameters that in turn pass on some type of payload to a service accross the wire. I've built Web APIs that accept images and endpoints that process files to produce custom HTTP response messages as required for integration.

If you've shared in any of these adventures, I have good news, friends: just like me, you've been training to write bots.

>> "This is going to be awesome." - me, giggling in the Cognitive Services AI Immersion Workshop at //Build 2017

The further I dive into the Bot Framework and the other complimentary services to support it, the more I feel at home. The types of things I have to do to build a bot feel natural, like things that I've been doing for years. The SDKs are well thought out and improving. Lessons learned in other areas of SDK and API design have been applied and are evolving and being tailored to meet the specific needs of interacting with people using conversation. And the cloud enables some very interesting capabilities without the effort that would have been required just a few short years ago.

So, yeah...this is going to be awesome.

## So, What the Heck is a Bot?

A bot is not the arrival of sentient beings. A bot itself is not an implementation of a neural network or the birth of a T-1000.  You'll hear repeatedly that as a bot developer, your job is not to pass the Turing test or even allow users free-form interaction.

From a technology perspective, a bot is nothing more than an API endpoint that accepts data and returns a response. The messages that are sent by, received to and proxied through the Microsoft Bot Framework are JSON, which makes them human-readable and easy to log and debug.  

From the view of a delivered product, a bot puts some tools in place to allow your customers or staff to access business intelligence or issue commands relevant to your business' needs.  It will interpret what the user is saying, look data up in your data stores, make decisions through logic you create, perform tasks on behalf of your users or the system and ultimately help a user accomplish a task.

You will be responsible for hosting the bot in some way. Because it's just a service that you likely already understand, you can stuff it on an existing web server or ship it out to the cloud. And because bots can leverage other cloud and programming features you'll find that you're already well-equipped with an arsenal that will help you build some pretty powerful experiences.

## The Microsoft Bot Framework, As Described by a Simpleton

I am not going to go too deep into the tech here. There are some great resources out there that do so, but the important bits for a developer are what I refer to as the "lifecycles" of the elements of which the bots are composed.  There are also many ways to build and host bots, so I'll assume you're going to go with the Visual Studio experience, with configuration and deployment destined for Azure. As a 'dotnet' developer I am also going to focus my efforts in the c# domain, though there are kits available for other languages including node and Java.

A bot is typically composed of:
 - Some sort of UI
 - A channel that allows communication between the user and the bot
 - An API that you host that is configured as the endpoint for a channel
 - A JSON payload with user input, payload and channel-specific information
 - System messages that help you control flow
 - State to help you manage conversation
 - The Bot Framework, to help turn conversation payload into something interesting to your application code, and
 - Logic you write to respond to the incoming messages.

The entry point for any bot is the user interface (there is an exception to this which we'll cover in detail, but lets start here). The UI is something that you can write yourself (such as your web page) but it is just as likely or more so that your user will come in though UI that you didn't write. These are called channels and they exist in the form of Facebook Messenger, SMS, Skype and so forth. And herein lies the biggest value proposition of the Bot Framework: the abstraction over those channels.

User interaction is configured for each channel in the Azure portal. You will need to register your application with each of the respective third parties (such as creating an application in Facebook) and then setup application IDs, webhooks or other aspects to establish channel communication. 

The bot itself is a Web API project that accepts HTTP POST requests (note that the V3 API is written on .NET full framework, whereas the upcoming V4 is written for .NET Core). 

The incoming message from a channel - the user input - comes across the wire as JSON and goes through the model binding process as would any payload in your MVC or Web API project.  There are pre-defined types provided by the framework to help make this fairly painless.

In your API controller you are responsible for inspecting the type of activity that was sent from the channel and pushing the message to the appropriate code. This may mean that you've recieved text from a user, that someone has liked a response your bot sent, or a signal that a user has joined a conversation.

In the event of a non-system message, you're likely going to send the activity on to a dialog class that will manage the conversation with the user. The purpose of each dialog is to segregate related bits of code that would help to form a user experience. The typical pattern is to create a "root" dialog from which all conversations can be managed. From the root, you guide users by using suggestions or buttons so that they can move in a direction that helps them find success. 

So, this is where it gets interesting (and more than a little bit complicated).


## You Can't Build a Perfect Bot, So Don't Try (From the "Personal Experience" files of James)

While your job is to create an experience (dialog) or set of experiences (dialogs) that will help the user get at what they need to, some users are hell bent on failure. Well, at least that's what it looks like when you're reviewing the logs.

> when r u open and is there chocolate pie. it's for Wedsnday. it's my aunts birthday and i haven't seen her in a long time and I think she likes it. or we could have a cake if you have some  - Actual Facebook message



## Chosing a Channel
In conversation with other bot writers over the last little while the conversation keeps coming up around how they chose the channels they use in their bot. There is no magic recipe for which channels to use for your organization, but in my case the primary channels will be SMS and Facebook and later via our web site.  For you, the choice will likely come down to the way your users already behave. For one colleague, this meant using Skype as an internal knowledge bot. For another, it was a combination of SMS and web. 

Here's why we chose to move ahead with our selection of SMS and Facebook:

 - Our bot users are mostly customers. When they place orders we proactively start conversations with them. With the conversation primed and started on our end, we can know with greater certainty how to guide the user into the pit of bot success.
 - Some of our users initiate conversations with us by texting our company phone number. By using the SMS channel we can do a "soft" identification with the channel data, resolve the customer and give them information about upcoming orders.
 - Most of our communication is done with customers on Facebook. Approximately 80% of inquiries are about product information and store hours and location. As our Social Media presence grows having a bot to handle this 80% is an effective way to keep our labor costs down for questions like, "When will you have Schmoo Tortes again?" or "Do I have a pie ordered for next Thursday?" or even, "When will you be selling that bread you have with roasted potatoes and caramelized onions?"










