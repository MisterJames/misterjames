title: How to deploy to Azure Functions using GitHub
layout: post
tags:
  - Azure Functions
  - c#
  - 5 days of Azure Functions
categories:
  - Development
authorId: james_chambers
featureImage: logo_579.png
originalUrl: 'http://jameschambers.com/'
date: 2016-11-16 18:02:07
---

When we start getting serious about using Azure Functions, we're likely going to want a little more horsepower to edit them. Visual Studio gives us a top-flight editing experience, but how do we work with Azure Functions locally? And how do we deploy them to the cloud?


When we use GitHub to store our code, we enable an easy way to deploy our functions continuously to Azure.

<!-- more -->

{% include_markdown _includes/Five-Days-Overview %}

## How This Works





## Benefits
This is pretty handy. It means that you can have a very rapid-to-deploy path that gives you the ability to build things quickly and get them out to Azure completely hands off.

Using a source control as a trigger for deployment will also help encourage your team to avoid using the portal to edit their scripts (there's a related caveat to this below). This is great because it means that no one will make unsolicited or accidental changes to the scripts in your portal...the UI is locked down to prevent out-of-band updates.

Because you can pick a branch to deploy from, you can actually use branches 

## Caveats
Ah yes, 'tis true that this ease of deployment comes at a cost. I would like to point out that in this scenario we are using the Kudu build and deployment pipeline. For those of you who practice CI/CD in such a way that the assets from your build server are promoted through each of your environments, this is not the correct path for you.

Because each merge to a branch is going to trigger a build for the environment that is watching it, you're actually getting different builds going to each environment. This isn't entirely a loss; after all, we're supposed to be entirely environment agnostic, right? If we don't care about the operating system or the machine that is running it, and the exact same bits are used to build it each time, do we have a problem?

Well...some folks (including the very well respected crew over at [ThoughtWorks](https://www.thoughtworks.com/insights/blog/enabling-trunk-based-development-deployment-pipelines)) consider a branch per environment to be an anti-pattern.  I couldn't agree more when we're talking about traditional software, architectures and environments (for oh, so many reasons), but in the world of PaaS, is it something that we should be rethinking?

But, to counter to that the build and deployment bits are built on Kudu, which has an API that you can consume as part of your deployment pipeline. You can also use publishing profiles and msdeploy or wawsdeploy to get your functions out there with just a little bit of script. So, there's no reason to back away from Azure Functions if the idea of deploying from source control or per-environment branches are outside of your comfort zone. 

One final and important caveat is that your source control repository may contain far more than just the Azure Function code. Deploying _All The Things_ may not be in your best interest at best, and at worst could cause hard-to-triage problems out in the cloud.  And, because you don't have the ability to make iterative code changes in the Functions editor, your only way to make changes is to edit the code locally and push it up to GitHub.

## Other articles in this series
{% include_markdown _includes/Five-Days-Menu %}

Happy coding!  


