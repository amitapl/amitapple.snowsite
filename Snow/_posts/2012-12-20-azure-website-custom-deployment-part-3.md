---
layout: post
title: Customizing site deployment based on site's app settings in Azure Web Apps (Websites)
category: Azure Websites, Azure Web Apps
url: /post/38419111245/azurewebsitecustomdeploymentpart3
series:
	name: WAWSCustomizeDeployment
	current: 3
	part: Part 1 - Introduction to custom deployment
	part: Part 2 - Custom script generator
	part: Part 3 - Custom script generator
---

**In** previous posts I talked about [Microsoft Azure Web App's custom deployment feature](/post/38417491924/azurewebsitecustomdeploymentpart1) and the [tool to easily generate a deployment script](/post/38418009331/azurewebsitecustomdeploymentpart2), in this post I'll do a step by step guide on writing a custom deployment script.

Let's say we have two websites in azure, one is a node.js website and the other is an mvc4 web application, both sites sources are in the same repository so we need a deployment script that handles differently based on which website it is deploying.

[Full sample repository](https://github.com/amitapl/CustomScriptSample)

Some Prerequisites:

1. Assuming git is installed and the websites are in a local git repository.
2. Install [node.js](http://nodejs.org/).
3. Install [azure-cli](http://www.windowsazure.com/en-us/manage/linux/other-resources/command-line-tools/) by 
running the following command: `npm install azure-cli -g`


### (a) First let's generate a deployment script for the node.js website:
- Go to the root of the repository.
- Enter the following command: `azure site deploymentscript --node --sitePath nodejs`
- Where "nodejs" is the path to the node.js website directory.

![](/images/2012-12-20-azure-website-custom-deployment-part-3.md1.png)

- Notice the files that were generated:
> - .deployment - a file telling which command to run for deployment (currently deploy.cmd).
> - deploy.cmd - the deployment script.
> - nodejs\web.config - configuration for iis to run node.js.
> - nodejs\iisnode.yml - this file allows some configuration settings relating to node.js, [more info on iisnode.yml](http://tomasz.janczuk.org/2012/05/yaml-configuration-support-in-iisnode.html "more info on iisnode.yml")

* Rename deploy.cmd to deploy.node.cmd: `move deploy.cmd deploy.node.cmd`

### (b) Let's generate a deployment script for the mvc4 web application:

At the root of the repository enter the command:

    azure site deploymentscript --aspWAP mvc4\Mvc4WebApplication\Mvc4WebApplication.csproj -s mvc4\Mvc4WebApplication.sln

![](/images/2012-12-20-azure-website-custom-deployment-part-3.md2.png)

* Rename deploy.cmd to deploy.mvc4.cmd: `move deploy.cmd deploy.mvc4.cmd`

*NOTE: You can also edit this generated file (deploy.cmd) with any custom steps you have, you can also test it on your machine simply by running it, it will publish your website to %REPOSITORY_ROOT%\artifacts.*

### (c) To decide which script should run, based on the website we are currently deploying, we'll use the "app settings" feature in Microsoft Azure Web Apps:

* Create a deploy.cmd file under the root of the repository with the following:

<script src="https://gist.github.com/4349245.js"></script>

* %SITE_FLAVOR% will be set by Windows Azure with the value we'll give it in the management portal.

* Add all generated files and commit them to the repository.

### (d) Now let's try to push our repository to our Microsoft Azure Web App:

    git push WA master

![](/images/2012-12-20-azure-website-custom-deployment-part-3.md3.png)

* We receive an error and the deployment fails since we still haven't set the app setting yet, so let's do that.

* Go to the website on windows azure management portal and add under the CONFIGURATION tab under "app settings" a setting with name SITE_FLAVOR and value nodejs/mvc3 (based on the current site we're configuring).

![](/images/2012-12-20-azure-website-custom-deployment-part-3.md4.png)

![](/images/2012-12-20-azure-website-custom-deployment-part-3.md5.png)

* Click on the "Save" button.

* Now we can either push our changes again (we'll need a new commit, even an empty one, otherwise it'll tell us that nothing has changed and the deployment won't reinitiate).
* Or we can go to the DEPLOYMENTS tab in Windows Azure portal, select the last deployment which failed and push the RETRY button to retry the deployment.

![](/images/2012-12-20-azure-website-custom-deployment-part-3.md6.png)

### That's it, now we have a working mvc4/node.js website

![](/images/2012-12-20-azure-website-custom-deployment-part-3.md7.png)

![](/images/2012-12-20-azure-website-custom-deployment-part-3.md8.png)

**NOTE: Another improvement we could do here is to store the repository on GitHub/Bitbucket and connect them to our 2 sites, now every time we push to GitHub/Bitbucket, both of our sites will be deployed.**

The repository I've used can be cloned from [here](https://github.com/amitapl/CustomScriptSample).
