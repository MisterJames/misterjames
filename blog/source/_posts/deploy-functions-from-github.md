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
date: 2016-11-17 07:02:07
---

When we start getting serious about using Azure Functions, we're likely going to want a little more horsepower to edit them. Visual Studio gives us a top-flight editing experience, but how do we work with Azure Functions locally? And how do we deploy them to the cloud?

![](https://jcblogimages.blob.core.windows.net:443/img/2016/11/deploy-from-git.png)

When we use GitHub to store our code, we enable an easy way to deploy our functions continuously to Azure, but it doesn't come without caveats. This post is about getting you familiar with the benefits, side-effects and consideration points you'll need to make as you move towards continuous deployment in Azure Functions.

<!-- more -->

{% include_markdown _includes/Five-Days-Overview %}

## How This Works
Here is a high-level summary of how your code gets from GitHub (or any other source control service you configure) out to your Azure Function App:
 - You check in some code
 - A postback is consumed by a service in Azure
 - Your code is cloned to a local-to-Azure path
 - A deployment is created for you
 - Have some coffee

Let's have a look at a few of the details up close.

## Setting up Deployment from Source Control
There's a pretty easy trail to follow in the Azure Portal to get GitHub hooked up, with the one requirement being that you have a repository ready to go. Create the repo and push it to GitHub (or create it on GitHub) then navigate through the Functions Apps UI to "Function app settings" and then under "Deploy" click on "Configure continuous deployment" to sign in to GitHub and pick your repo.

But if you've already created a few functions in your app, you will want to get those down to your computer to work on them in Visual Studio and put them under source control. Here's how to do it:

 1. If you didn't create it locally, clone your repo down to your machine
 1. Download your functions from the Kudu portal (this will be a ZIP, see below)
 1. Unzip the functions archive and copy the functions into the root of your repo, there will be one folder per function
 1. Open the files in Visual Studio and hack away
 1. Commit and push your changes

In step 2 I mention downloading the wwwroot directory from the Kudu console. To do that, head back to "Function app settings" and then under "Deploy" click on "Go to Kudu". Drill into the `site directory and then click on the download icon to get your functions. 

![](https://jcblogimages.blob.core.windows.net:443/img/2016/11/download-functions.png)

You could also grab these files by using an FTP client and the credentials you have set up in the Azure Portal. 

## Benefits
This is pretty handy. It means that you can have a very rapid-to-deploy path that gives you the ability to build things quickly and get them out to Azure completely hands off.

Working locally has other benefits. It's much easier to work with multiple files without clicking around in a browser and hopping throughout your functions.

![Editing functions in Visual Studio Code](https://jcblogimages.blob.core.windows.net:443/img/2016/11/functions-in-vs-code.png)

Visual Studio and VS Code are powerful tools in our belts, and things like IntelliSense, a tabbed editing interface and the ability to work offline are all big wins.

Using a source control as a trigger for deployment will also help encourage your team to avoid using the portal to edit their scripts (there's a related caveat to this below). This is great because it means that no one will make unsolicited or accidental changes to the scripts in your portal...the UI is locked down to prevent out-of-band updates. Because you can pick a branch to deploy from, you can actually use branches for different environments. 

## Caveats
Ah yes, 'tis true that this ease of deployment comes at a cost. I would like to point out that in this scenario - deploying from a branch - we are using the Kudu build and deployment pipeline for each deployment target.  For those of you who practice CI/CD in such a way that the assets from your build server are promoted through each of your environments, this is not the correct path for you.

Because each merge to a branch is going to trigger a build for the environment that is watching it, you're actually getting different builds going to each environment. This isn't entirely a loss; after all, we're supposed to be entirely environment agnostic, right? If we don't care about the operating system or the machine that is running it, and the exact same bits are used to build it each time, do we have a problem?

Well...some folks (including the very well respected crew over at [ThoughtWorks](https://www.thoughtworks.com/insights/blog/enabling-trunk-based-development-deployment-pipelines)) consider a branch per environment to be an anti-pattern.  I couldn't agree more when we're talking about traditional software, architectures and environments (for oh, so many reasons), but in the world of PaaS, is it something that we should be rethinking? (I will talk about this in greater detail in another post).

One final and important caveat is that your source control repository may contain far more than just the Azure Function code. Deploying _All The Things_ may not be in your best interest at best, and at worst could cause hard-to-triage problems out in the cloud. You can likely get around this with an orpahned branch, but that feels akward to me. 

Finally, because you don't have the ability to make iterative code changes in the Functions editor, your only way to make changes is to edit the code locally and push it up to GitHub, thus triggering a deployment. I actually consider this a perk as well, but it's something you should be aware of.

## Alternate Deployment Possibilities
If GitHub to Azure direct isn't your thing, remember that the build and deployment bits are built on Kudu, which has an API that you can consume as part of your deployment pipeline. You can also use publishing profiles and msdeploy or wawsdeploy to get your functions out there with just a little bit of script.  

Using a script as part of another build server process also gives you the ability to extract the Function assets and deploy them seperately from the rest of your project code.

In short, there's no reason to back away from Azure Functions if the idea of deploying from source control or per-environment branches are outside of your comfort zone.

## Next Steps
While the jury is no longer out on why you'd want to have a build and deployment pipeline in place, there certainly can be early wins in a project for testing and prototyping by deploying a project directly from source control. For non-critical workloads and early in your prototyping cycles, deployment from GitHub or any other source control service may give you an easy way to get part of your app in front of clients and consumers. Give it a try and discuss it with your team to see if this feature has a place in your project. 

## Other articles in this series
{% include_markdown _includes/Five-Days-Menu %}

Happy coding!  


