---
layout: post
title: Custom Deployment Scripts For Windows Azure Website Using Git Deployment
category: Windows Azure Websites
url: /post/38417491924/azurewebsitecustomdeploymentpart1
series:
	name: WAWSCustomizeDeployment
	current: 1
	part: Part 1 - Introduction to custom deployment
	part: Part 2 - Custom script generator
	part: Part 3 - Custom script generator
---

The coolest feature on Windows Azure Websites is the ability to deploy your website using git.
Do a `git push` and bam you're done and deployed within seconds.
The deployment process is automated, the process will look at the files on the git repository and decide which kind of website it is (asp.net, node, ...) and based on that will do the required steps for the deployment.

For example for an MVC web app it'll find the solution file and determine which project is the actual web app project and with those it'll msbuild that project, the artifacts from the build will be placed in a temporary folder and only the files that were changed will be copied to the wwwroot location for the actual deployment.

The nice thing is that files that were removed will also be removed on the wwwroot location but only if they were actually deployed the previous time (so files that are generated on the fly on the wwwroot directory won't be removed).

So what if you want to customize the deployment process, for example you want to run your tests before deploying (or after) and cancel the deployment if they fail?

That's what the custom deployment feature is about, you just need to add a file to the root of your repository with the name `.deployment` and the content:

    [config]
    command = YOUR COMMAND TO RUN FOR DEPLOYMENT

this command can be just running a script (batch file) that has all that is required for your deployment, like copying files from the repository to the web root directory for example.

You also get some environment variables that you need in order to get and put things in the right place:

* DEPLOYMENT_SOURCE - The path for the root of your repository (In Azure).
* DEPLOYMENT_TARGET - The wwwroot path (the deployment destination directory).
* DEPLOYMENT_TEMP - Path to a temporary directory that will be removed after the deployment.
* MSBUILD_PATH - Path to msbuild executable.

With all of those you can create your own deployment script, a simple one for example:

    @echo off
    echo Deploying files...
    xcopy %DEPLOYMENT_SOURCE% %DEPLOYMENT_TARGET% /Y

This script will copy all of the files from your repository to the wwwroot directory.

But to make things easier, you can use the azure-cli tool which will actually generate a deployment script for you that will do exactly the same deployment process as the default one but now you are able to update that script and add (or remove) your own steps.

More on that in the next post:
[Custom Deployments Script Generator](/post/38418009331/azurewebsitecustomdeploymentpart2)
