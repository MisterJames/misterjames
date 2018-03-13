title: 'Choosing a Channel for the Microsoft Bot Framework'
layout: post
tags:
  - Bot Framework
categories:
  - Development
  - Chez Angela
authorId: james_chambers
originalUrl: 'http://jameschambers.com/'
date: 2018-03-10 20:16:13
---
Knowing where to tune in so that your users can more readily interface with you is a critical bot of your bot's success. 

![Tuning into your users](https://jcblogimages.blob.core.windows.net/img/2018/bakebot/2018-03-12-1720efd2.png)

Knowing quantitatively where your users congregate (and wish to interact) is the best way to understand how you can start to piece together your bot, but you don't have to have all the answers right away.

<!-- more -->

## Channels for Your Bot

Your users will be approaching your business from different mediums all the time. Some users might use Facebook, others Skype, others still SMS. A channel in the Microsoft Bot Framework is a funnel through which your users will communicate with your bot code.

The problem is, should you try to write a bot on your own, is that nearly every single source of user interaction out there will define its own standard for bot interaction. These are likely going to be JSON in some kind, with an end point that you can post data to and a corresponding web hook that you'll need to write to catch responses and other input. They will require some form of identification. They will almost certainly have a custom payload that is unique to their platform, and distinct, often unique ways to render information to the end user.

The framework does a pretty good job of aggregating that data into a common set of objects that are easy to code against. This makes it a little easier to write shared services and renderers that will help you to craft a unique experience.

The currently supported channels are:
 - Bing
 - Cortana
 - Email
 - Facebook
 - GroupMe
 - Microsoft Teams
 - Skype / Skype for Business
 - Telegram
 - Twilio
 - Web Chat

The Direct Line protocol also allows you to create a custom client application and integrate with the Bot Framework using a fairly straightforward control in a number of different scenarios, with or without front-end frameworks like React.

Note that each channel will give you certain capabilities that may not be offered in other channels, including access to specific user data or the way that you can render output and controls to interact with your bot. You need to take these into consideration so that you can manage your output and flow appropriately on each channel.

## Choosing a Channel
In conversation with other bot writers over the last little while the conversation keeps coming up around how they chose the channels they use in their bot. There is no magic recipe for which channels to use for your organization, but in my case the primary channels will be SMS and Facebook and later via our web site.  

For you, the choice will likely come down to the way your users already behave. For one colleague, this meant using Skype as an internal knowledge bot. For another, it was a combination of SMS and web. 

Here's why we chose to move ahead with our selection of SMS and Facebook:

 - Our bot users are mostly customers. When they place orders we proactively start conversations with them. With the conversation primed and started on our end, we can know with greater certainty how to guide the user into the pit of bot success.
 - Some of our users initiate conversations with us by texting our company phone number. By using the SMS channel we can do a "soft" identification with the channel data, resolve the customer and give them information about upcoming orders.
 - Most of our communication is done with customers on Facebook. Approximately 80% of inquiries are about product information, store hours and location. As our Social Media presence grows having a bot to handle this 80% is an effective way to keep our labor costs down for questions like, "When will you have Schmoo Tortes again?" or "Do I have a pie ordered for next Thursday?" or even, "When will you be selling that bread you have with roasted potatoes and caramelized onions?"

These reasons likely won't be right for you, but it may be worth starting to record some metrics to help you make a choice. Knowing that 81.4% of conversations on Facebook were about store hours, location and product info meant that we could make a safe assumption about how to best serve our customers. 

## Tuning In To Your Users

You may find that your users have different objectives based on their channel, or perhaps the channels are so different that they can't possibly deliver a consistant experience. This was the case with BakeBot. SMS users (plain text on phones) and Facebook (rich, visual experiences with multiple options for display in browser or in a dedicated messaging app) have very different expectations. 

We decided to start those conversations off very differently, so we have multiple `RootDialog` classes that allow us to model those experience appropriate to the channel in question.  This allows me to approach the handoff to the dialog in an explicit way. While not displayed below, I also have the ability to check and act on channel-specific data (like Facebook's "Get Started" command) which we'll look at in a different post.

```
  if (message.ChannelId == Channels.Facebook )
  {
      // do channel-specific things here...
      
      await Conversation.SendAsync(activity, () => new FacebookRootDialog());
      return Request.CreateResponse(HttpStatusCode.OK);
  }

  if (message.ChannelId == Channels.Sms)
  {
      await Conversation.SendAsync(activity, () => new SmsRootDialog());
      return Request.CreateResponse(HttpStatusCode.OK);
  }

  if(message.ChannelId == Channels.Emulator || message.ChannelId == Channels.WebChat)
  {
      await Conversation.SendAsync(activity, () => new GeneralRootDialog());
      return Request.CreateResponse(HttpStatusCode.OK);
  }

  reply.Text = "Hi, I'm BakeBot! I can't do much yet, but I'm learning. I'll let my humans know that you tried to reach us!";
  await connector.Conversations.ReplyToActivityAsync(reply);

  // notify humans...

  return Request.CreateResponse(HttpStatusCode.OK);
```

You'll notice that I have a fallback to channels I'm still okay with (namely the emulator and the web chat channels) and then finally I reply with a generic message and continue on with a notification to a human.

## Wrapping Up

Okay, next up we're going to hop into some bot code, so make sure you have your pre-requisites in place and get ready to start bot-making.

Happy coding!

{% include_markdown _includes/bots-overview %}
