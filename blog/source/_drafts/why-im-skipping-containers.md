layout: post
tags:
  - Visual Studio 2015
  - git
categories:
  - Development
authorId: james_chambers
originalUrl: 'http://jameschambers.com/'
date: 2016-03-28 22:12:54
---

## These Aren't The Solutions You're Looking for
We're going to someday look back at containers and say they were an instrument of change. They forced us to think about what we need from our servers in smaller slices. They encouraged and enabled patterns that were previously harder to accomplish. They allowed applications and services to focus on continued development while pushing the concerns of the environment further away from the software proper. 

They are allowing legacy applications to 

But they are not where I will be focusing  

Thanks, But No Thanks, Docker

## Migration Is the On-Ramp, Not the Destination
"Lift and shift" is just an entry point to the wider cloud offerings. There is a lot more potent stuff out there, and starting to use virutalization of some sort is likely just the beginning of your migration to cloud. You can't claim to have succeeded in moving to the cloud if all you're doing is running your software on virtual machines. VMs can be a great way to get started, but they _by no means_ reflect a mature implementation of cloud architecture.

So, where do you go beyond that?  I believe that the answer is in the Platform as a Service offering, be it on AWS or Google's cloud or Azure. PaaS architecture allows organizations to focus on the solution rather than the infrastructure. It's not that it doesn't matter - the greatest painters knew what materials they were painting on and understood their qualitites - but infrastructure can fade into the background today much like the selection of a painter's canvas. 

## Say "No" to Snowflakes
There's a phrase that's becoming popular right now that goes something like, "Your servers shouldn't be pets with personality, they should be treated like cattle."  Well, I'm not building a farm, so you can keep your cows.

But, listen, as far as servers go, yes, I 100% back the effort to remove the unique snowflake that is oh-so-very special in your organization. A named server is a liability for too many reasons and you should work to expunge them from your network if you aren't in a position to leverage PaaS.

From this push away from those snowflakes has been the push to script out our environments and start thinking of the virtualized resources we rely on as part of our code base. This is a significant improvement over where we were. Servers can be spun up without deploying humans. We can establish desired state for scenarios where the operating system and features we need aren't available on an existing container. 

But rather than thinking about infrastructure as code, we should be thinking more abstractly in our patterns and approaches. The tools are here, we need to start leveraging them.  From this day forward, I hope to never have to start a project where my regular duties might include installing IIS or scheduling operating system updates. 

## Diving In Deeper than the Platform



## Architecture vNext

I had the great opportunity to chat with some great folks on Azure Functions recently, and where how 


