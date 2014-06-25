---
layout: post
title: How to deploy Azure WebJobs
category: Azure Websites, Azure WebJobs
url: /post/74215124623/deploy-azure-webjobs
---

**Windows Azure WebJobs** is a new feature coming from Windows Azure Websites, you can read all about it [here](http://www.windowsazure.com/en-us/documentation/articles/web-sites-create-web-jobs/).

While you can easily add a new WebJob using the Windows Azure portal, you may want to deploy your WebJob in other ways (ftp / web deploy / git), In this post I'll explain how these WebJobs are stored on your Azure Website and how you can deploy a new WebJob.

## Where is your WebJobs stored? ##

A WebJob is stored under the following directory in your site:

`site\wwwroot\App_Data\jobs\{job type}\{job name}`

Where {job type} can be either **continuous** for a job that is always running or **triggered** for a job that starts from an external trigger (on demand / scheduler).

And {job name} is your WebJob's name.

So a continuous WebJob called myjob will be located at:

`site\wwwroot\App_Data\jobs\continuous\myjob`

## What should be inside a WebJob directory? ##

The WebJob directory can contain 1 to as much as you'd like files but at the least it should contain the script that starts the WebJob's process, this script can currently be: batch (.exe/.cmd/.bat), bash (.sh), javascript (.js as node.js), php (.php) or python (.py).

The script to be run is automatically detected using the following logic:

* First look for a file called run.{supported extension} (first one found wins).
* If not found look for any file with a supported extension.
* If not found, this is not a runnable WebJob

> **NOTE:** If you have some other type of execution engine you wish to use and is currently not supported, you can always create a run.cmd file and create your executor command there (`powershell -Command run.ps`).

## Deploy my WebJob ##

So with this information we know that in order to create a new continuous job called myjob all you have to do is get your job's binaries to that folder.

One way to do this is to connect to your site via ftp, create the right directory and copy the binaries there (should include at least one supported script file).

That's it, the WebJob will be auto-detected and immediately start running.

## Deploy Website + WebJobs ##

To deploy a website with WebJobs, all you'll need to do is to make sure you deploy your WebJobs are in the right place, take a look at the following structure as an example for a node.js site with a web job:

    ./server.js
    ./App_Data/jobs/continuous/myjob/run.cmd

While this project contains only 2 files, it actually is a website with a continuous WebJob, and you can use whatever deployment preference you have to deploy this (ftp / web deploy / git / ...).

## Re-deployment ##

**Continuous** - After you redeploy a WebJob, the currently running process will abort and restart with the new binaries.

**Triggered** - Redeployment will not affect a currently running WebJob but the next run will be using the new WebJob's binaries.

> **NOTE:** Before a WebJob starts to run, the binaries for it are copied to a temporary directory, this way you can always re-deploy a WebJob safely without worrying that the files are locked.

## Triggered WebJob caveat ##

One issue we encounter with this method is that when you deploy a triggered WebJob, the result is a WebJob that is only **on demand** meaning you need to press the **RUN ONCE** button in the portal in order to initiate it and (for now) there is no way to easily add a schedule to it.

A workaround would be to create a dummy schedule WebJob with the name you are about to deploy and the deployment will just replace the binaries but keep current schedule.
