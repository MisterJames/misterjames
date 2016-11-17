title: 'Accept Stripe Payments in ASP.NET Core'
layout: post
tags:
  - Visual Studio 2015
  - Stripe
categories:
  - Development
authorId: james_chambers
originalUrl: 'http://jameschambers.com/2016/04/accept-stripe-payments-in-aspnet-core/'
date: 2016-04-29 21:12:51
---


![Stripe Payments in ASP.NET Core](https://jcblogimages.blob.core.windows.net/img/2016/04/github-auth.png)

Let's have a look at what it takes to allow users to authenticate in our application using GitHub as the login source, and you can check out the Monsters video take of this on [Channel 9](https://channel9.msdn.com/Series/aspnetmonsters/Episode-26-GitHub-Authentication-in-ASPNET-Core).

<!-- more -->

## Background and Overview


In short, the steps are as follows:

 - register with Stripe
 - create a plan
 - get api keys
 - get user secrets set up
 - install nuget package
 - create models to support proper tracking
 - add migration / update database
 - add controller
 - add view

## Next Steps

 1. 

Finally, check out the Monsters' video on Channel 9 where I walk through the code for this live.

<iframe src="https://channel9.msdn.com/Series/aspnetmonsters/Episode-26-GitHub-Authentication-in-ASPNET-Core/player" width="640" height="360" allowFullScreen frameBorder="0"></iframe>

Happy Coding!