---
layout: post
title: Getting notified when your Azure triggered WebJobs completes
category: Azure WebJobs
---

**Microsoft Azure WebJobs** are awesome, and now a little bit more... I'm going to show you how you can setup a notification whenever your triggered (on-demand or scheduled) WebJobs completes.

The notification part is done by integration between Azure and Zapier which provides many different notification types such as: email, phone call, SMS, Facebook post and more, for this post I'll use a phone call but it is very easy to use any of them.

> NOTE: In a previous [post](http://blog.amitapple.com/post/56390805814/deployment-email/) I explained about [Zapier](http://zapier.com) and how you can have a notification when your Azure Web App deployment completes, this is very similar only with a triggered WebJob.

**Let's do it:**

### Prerequisites ###

- An Azure Web App with at least 1 triggered (on-demand or scheduled) WebJob (although you can add it later).

- Sign up to [Zapier](https://zapier.com/app/signup)

- Have both the Zapier and Azure portal open

### Steps ###

- Go to Zapier and create a new zap (*Make a Zap!*).

- For the trigger service select **Azure Web Sites**.

- For the trigger select **New Triggered WebJob Run**.

- For the action, we'll select **Phone** and **Call Phone** for this sample but any can be selected.

  ![](/images/webjobs_phone1.png)

- Click Continue

- We need to connect to our Azure Web Site hosting our triggered WebJob, for this we need one piece of information from the Azure portal.

**This is the tricky part:**

- If your website has continuous deployment setup --> in the Azure portal go to your website, click on the **CONFIGURE** tab and under the **git** section copy the url which is under the **DEPLOYMENT TRIGGER URL**.

  ![](/images/2013-09-06-deployment-email.md3.png)

- If you don't have continuous deployment, you can author this url yourself, it is: `https://{userName}:{password}@{siteName}.scm.azurewebsites.net/deploy` where you get the `{userName}` and `{password}` from your site's publishing profile.

  ![](/images/webjobs_phone3.png)

  ![](/images/webjobs_phone4.png)

- Go back to the Zapier site and paste this url to the **Deployment URL** textbox, enter a name for this website account and click continue.

  ![](/images/webjobs_phone2.png)

- Now create your phone account by providing the phone number and verifying it.

- At this point you can filter when you actually want to initiate the action, for example only when the WebJob run fails or only for a specific WebJob (by name), for now we keep this empty as we want to be notified on all WebJobs runs, click continue.

- Next you specify the content of the message, it can be static and dynamic using the WebJob run result.

- For example we'll use: `Hello the WebJob named {{job_name}} has completed with status {{status}} and took {{duration}}`, on the right you can use the "Insert fields" button to add other interesting dynamic fields.

- You can even choose the voice of the caller (Man/Woman), I'll let you pick this one.

  ![](/images/webjobs_phone5.png)

- Continue

- **Test this Zap** lets you test your zap by getting previous WebJob runs and doing the selected action on them, click the button and then you can skip the step or test your Zap.

- **Name and turn this Zap on**

- Now go to your Azure portal, run your WebJob, wait for it to complete and wait for the call :)
 
Get more help about [Windows Azure Web Sites on Zapier](https://zapier.com/help/windows-azure-web-sites/).
