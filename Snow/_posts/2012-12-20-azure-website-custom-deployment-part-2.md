---
layout: post
title: Windows Azure Websites - Custom Deployment Scripts Generator
category: Windows Azure Websites
url: /post/38418009331/azurewebsitecustomdeploymentpart2
series:
	name: WAWSCustomizeDeployment
	current: 2
	part: Part 1 - Introduction to custom deployment
	part: Part 2 - Custom script generator
	part: Part 3 - Custom script generator
---

**With** Windows Azure Websites you can deploy your website by simply pushing your git repository, this will automatically deploy your website, and if you want to control this deployment flow you can use the custom deployment feature.

To make it easy on us there is a custom deployment script generator feature in the azure-cli tool that will simplify the whole process, basically it will generate a script that has the same logic as the automatic deployment one only now you can change it and also run it locally to test it.

All you have to do is:

a. Install [node.js](http://nodejs.org/), this is required to run the azure-cli tool.

b. Install the [azure-cli](http://www.windowsazure.com/en-us/manage/linux/other-resources/command-line-tools/) tool, it'll also give you some cool features on managing azure related resources directly from the command-line:

    npm install azure-cli -g

c. Go to the root of your repository (from which you deploy your site).

d. Run the custom deployment script generator command:

    azure site deploymentscript [options]

You can find help on the [options] by using the -h flag.

For example to create a script for deploying an asp.net web application you do the following:

    azure site deploymentscript --aspWAP pathToYourWebProjectFile.csproj -s pathToYourSolutionFile.sln

Or for deploying a node website:

    azure site deploymentscript --node

Any of these commands will generate the files required to deploy your site, mainly:

> * .deployment - Contains the command to run for deploying your site.
> * deploy.cmd - Contains the deployment script.

Now you can edit the deploy.cmd file and add your custom steps.

Looking at the file you'll notice that it may look a bit complicated at first but actually most of the code there is to make sure the script can run locally so the place to look for the deployment logic is here:

    ::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
    :: Deployment
    :: ----------

    ... [deployment steps]

    ::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

It'll contain very few steps and that's the right place to put your custom steps, for example: running your tests.

In this script you'll also notice the use of a tool called [KuduSync](https://github.com/projectkudu/KuduSync), this tool is the publishing tool which copies the website from the repository (or temporary directory) to the wwwroot directory.

The genius of this tool is that it'll only copy the files that were changed, it'll also remove the files that don't exist on the source but only those that were previously on the source so it won't remove any runtime generated files (it does that by keeping a files list of each deployment, this is what the manifest files are about).

After adding your own logic to the deployment script you can run in locally and test it to make sure it does what you need it to, it'll publish your site to a sub-directory called *artifacts* so make sure you're not add the files there to your repository.

When the script is tested, add the generated files to your repository (.deployment and deploy.cmd, for node also web.config and iisnode.yml) and push your repository to your windows azure website and see your custom deployment running.

[Next post a step by step on a specific custom script](/post/38419111245/azurewebsitecustomdeploymentpart3).
