---
layout: post
title: Running an OWIN web application on Azure Web Apps
category: Azure Websites,Azure Web Apps,OWIN
---

Some things that might be obvious to some people are not obvious for everyone, in this case I tried to figure out if and how OWIN works in Azure Web Apps (Azure Websites). As I used to work on the Azure Web Apps team I should know this kind of things, well I actually didn't.

So I tried looking online for how to host OWIN on Azure Web Apps and actually couldn't find anything.

I also want to mention that I also couldn't find a visual studio template for a plain OWIN web app without all the mvc/web api/jquery/... clutter.

In this post I want to address both these topics and help devs like me who couldn't find an answer.

## Hosting OWIN on Azure Web Apps ##

Well actually, it is very easy to host an OWIN web application on Azure Web Apps, you just need to use the IIS hosting method and it works.

* Install `Microsoft.Owin.Host.SystemWeb` nuget package
* Add the following line of code at the top of your `Startup.cs` file:

        [assembly: OwinStartup(typeof(Startup))]

Yes it so easy that it seems obvious, well now I know :)


## Clean OWIN Projects ##

Next I wanted to provide some clean startup
