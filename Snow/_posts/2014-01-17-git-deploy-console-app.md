---
layout: post
title: Git deploying a .NET console app to Azure using WebJobs
category: Windows Azure Websites
url: /post/73574681678/git-deploy-console-app
---

There's a new concept coming from Windows Azure Websites called **WebJobs** where you can have scripts running on a schedule, continuously or simply by invoking them manually.

In this article I'll talk about a simple way of deploying a .NET console application to Windows Azure Websites which will be deployed as a **continuous job**.

In short, a continuous job is treated similar to a windows service, as long as it's started/enabled, Azure will make sure it's up, it'll start the process (on all of your instances) and whenever the process goes down it'll simply bring it back up (after a 60 seconds delay).

The executable that is deployed as a continuous job should have some kind of an infinite loop (`while (true)`).

> **Note:** While you can experiment with continuous jobs in free or shared websites, it'll only work properly in a standard website that has the "always on" setting set, this is since in free/shared sites the jobs process will be brought down after about 20 minutes of no requests to it (a request to the jobs process can be to check the status of current jobs).


Enough chit-chat, now to the main event, we have .NET code which we want to (continuously) run on Azure, here are the steps:

* **Write** your .NET code as a .NET console application, here is a sample:

<script src="https://gist.github.com/amitapl/8467381.js"></script>

* **Add** your code to a git repo.

 `git init`

 `git add .`

 `git commit -am Coding`

* **Create** a new web site with source control.

![](https://31.media.tumblr.com/ec6583e81f55d0d6915a3bafbdb43ea4/tumblr_inline_mzixv5U1is1rvdhx0.png)

* **Get** the url to your site's git repository

![](https://31.media.tumblr.com/65edce99bcca99c1b7bc585e65b0046d/tumblr_inline_mzixy34iGL1rvdhx0.png)

* **Push** the repository to your site.


 `git push http://.../.git master`


* That's it, you now have your .NET console application running on Windows Azure Websites, just go to the WEBJOBS tab and take a look.

![](https://31.media.tumblr.com/322e8c33d777d4df4abe0d75526f9aa8/tumblr_inline_mziy3owv9K1rvdhx0.png)


> **Note:** Don't forget to enable the **Always On** feature if you're on standard to make sure this job will never stop running.


Now to see the continuous job logs click on the "logs" link (you'll need to enter your publishing user name and password, if you're on chrome even better - it just works), there you would see all the system logs (when the job started, stopped, etc...).

To see the logs produced by the application (console output and error) you need to enable application logging under the CONFIGURATION tab, your logs can go to your file system/azure storage table or blob.


> **Note:** You can also use app settings/connection strings which would come from the site's configuration, same as you would do for your ASP.NET website.


There are more awesome things to say about webjobs, but that's in next posts.

You can use my repo to try it: [](https://github.com/amitapl/ContinuousHelloWorld)
