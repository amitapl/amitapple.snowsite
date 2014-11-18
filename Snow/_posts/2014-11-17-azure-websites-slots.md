---
layout: post
title: Azure Websites Deployment Slots - Explained
category: Azure Websites, Deployment Slots
---

One of the premium features you get when using Azure Websites in a standard SKU is the **deployment slots** feature also known as **staged deployment** but it is actually more than that.

In this post I will go over the **deployment slots** concept and what you can do with it.

![Deployment slots](/images/slots1.png)

## What are those deployment slots? ##

From a (standard) website you can create deployment slots which will actually be Azure Website instances that are tied to that Website.

A deployment slot will carry the name of the Azure Website + the name of the slot, for example:

If my Azure Website is called **mysite** and I create a slot called **staging** then my slot will be an Azure Website with the name **mysite(staging)** and its url will be **http://mysite-staging.azurewebsites.net**.

![Add deployment slot](/images/slots2.png)

> It's important to emphasize that the slot is in itself a regular Azure Website, it will have its own app settings, connection string, any other configuration settings and even an scm site (**https://mysite-staging.scm.azurewebsites.net**).

> In fact by default each Azure Website has a single deployment slot called **production** which is the Azure Website itself.

You can add more than one **deployment slot**.

## Why do I need this? ##

The first feature of deployment slots is the **Swap Slots** and it's used for **Staged Deployment**

![Add deployment slot](/images/slots5.png)

In short, the **Swap** operation will exchange the website's content between 2 deployment slots.

> Later I'll explain what is swapped and what is not but note that swap is not about copying the content of the website but more about swapping DNS pointers.

So in our scenario we have the **Production** site with `index.html` that starts with `Hello World` and our **staging** slot has the same `index.html` but it starts with `Yello World`.

Before swap - **http://mysite.azurewebsites.net/index.html** will return `Hello World...`

After swap - **http://mysite.azurewebsites.net/index.html** will return `Yello World...`

Now to get this into a real life scenario.

### Staged Deployment ###

Deploying your website in the traditional way, whether deploying via WebDeploy, FTP, git, CI or any other way, has weaknesses that may or may not concern you:

* After the deployment completes the website might restart and this results in a cold start for the website, the first request will be slower (can be significant depending on the website).
* Potentially you are deploying a "bad" version of your website and maybe you would want to test it (in production) before releasing it to your customers.

This is where **staged deployment** comes into play. Instead of deploying directly to our production website we create a **deployment slot** used for **staging** and we deploy our new bits there.

Then we "warm" our site (**staging** slot) by making requests to it and we can start testing our new bits verifying everything works as expected. Once we're ready we hit the Azure Portal's **Swap** button (or PowerShell/xplat cli command) and the slots will be swapped.

Our customers will not hit the "cold start" delay and we have more confidence in our new bits.

### Auto-Swap ###

Since we want to test our website before going into production we have this manual step where we hit the **Swap** button to swap.

But if we only want to address the "cold start" delay we can configure the **Auto Swap** feature where the website automatically swaps a configured slot (in our case **staging**) with the **Production** slot after the deployment completes.

> Currently **auto-swap** only works when deploying using WebDeploy (deploying through VS will usually use WebDeploy) and Continuous Integration (VSO, GitHub, Bitbucket).
> FTP and `git push` will not cause an **auto swap**.

> Auto-swap can take a while to swap (1-2 minutes), until the swap completes any other attempts to deploy the website will fail.

To set this up you'll need to use the [Azure PowerShell tool](http://azure.microsoft.com/en-us/documentation/articles/install-configure-powershell/) ([download](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409))

In PowerShell use the following command:

    Set-AzureWebsite -Name mysite –Slot staging -AutoSwapSlotName production

This command will set Azure Websites to auto swap the **staging** slot into **Production** slot whenever **staging** is deployed.

> You can use the operation logs in the (current) Azure portal to see the **auto swap** operation status.


### Deployment Slot App Settings / Connection String / Configuration ###

One important concept to understand about **deployment slots** is how the configuration works.

A deployment slot is a full Azure Website and as one it has all the same configurations as any Azure Website. When you swap deployment slots there are some settings you actually need to keep with the slot and not swap them.

A setting that is not swapped is referred to as a setting that is **sticky to the slot**.

Some of the default settings that are **sticky to the slot**:

* Most obvious one is the url - **http://mysite-staging.azurewebsites.net/** will always point to the **staging** slot.
* **WEBSITE_HOSTNAME** environment variable for the **staging** slot will always be **mysite-staging.azurewebsites.net** and this is something we can use in our website code to find it's currently running in the **Production** slot or **staging** slot.
* Deployment settings - if you have the deployment profile for the **staging** slot, after a swap the profile would still point to the **staging** slot.
  > This also includes continuous integration settings - if you hooked your **staging** slot with a GitHub repository after a swap the hook will still exist between GitHub and the **staging** slot.

App settings and connection strings are **not** sticky to the slot and will remain with the website when swapped but we can configure selected app settings and connection strings to become **sticky to the slot** using a PowerShell command (not yet supported by the Azure portal).

Use this command in Azure PowerShell to set 2 app settings as **sticky to the slot**

    Set-AzureWebsite -Name mysite -SlotStickyAppSettingNames @("myslot", "myslot2")

And this command to set 2 connection strings as **sticky to the slot**

    Set-AzureWebsite -Name mysite -SlotStickyConnectionStringNames @("myconn", "myconn2")

> **Sticky to the slot** configuration is website-wide configuration and affects all slots in that website.
 
### Deployment Slots Traffic Routing ###

Another great feature for **deployment slots** is the **traffic routing** also known as **testing in production**.

This feature will allow you to route traffic that is coming to your Azure Website between your **deployment slots** based on percentage of the traffic.

This feature exists only in the new [Azure preview portal](https://portal.azure.com).

In the portal under your website there is a tile called **Testing in production**, click on it to get to the "Testing in production" *blade* where you can direct traffic coming to your website between all of your **deployment slots**.

![Testing in production](/images/slots4.png)

One usage scenario for this feature is [A/B testing](http://en.wikipedia.org/wiki/A/B_testing).

By default 100% of the traffic will go to the **Production** slot but you can create a new **deployment slot** with a slightly different version of your website (differs by what you want to A/B test) and add it there with a 50% value so 50% of your visitors will actually be served from the new slot.

Another scenario for this feature is having a **dev** slot that is a little less stable which gets 1% of the traffic so you can test feature currently being developed with real traffic.

[For more information on this feature](http://blogs.msdn.com/b/tomholl/archive/2014/11/10/a-b-testing-with-azure-websites.aspx).

## Wrap Up ##

I hope that if the **deployment slots** were just a mysterious link/tile/concept before, you now know how to master them as they can bring lots of value to your production website.
