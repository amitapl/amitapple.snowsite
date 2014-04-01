---
layout: post
title: Getting notified when your Azure triggered WebJobs completes
category: Windows Azure Websites, Windows Azure WebJobs
---

Microsoft Azure WebJobs are awesome, and now a little bit more...

I'm going to show you how you can setup a notification whenever your triggered (on-demand or scheduled) WebJobs completes.

The notification part is done by integration between Azure and Zapier which provides many different notification types such as: email, phone call, SMS, Facebook post and more, for this post I'll use a phone call but it is very easy to use any of them.

> NOTE: In a previous [post](http://blog.amitapple.com/post/56390805814/deployment-email/) I explained about [Zapier](http://zapier.com) and how you can have a notification when your Azure Website deployment completes, this is very similar only with a triggered WebJob.


**Let's do it:**

### Prerequisites ###

- An Azure Website with at least 1 triggered (on-demand or scheduled) WebJob (although you can add it later).

- Sign up to [Zapier](https://zapier.com/app/signup)

- Have both the Zapier and Azure portal open

### Steps ###

- Go to Zapier and create a new zap.

  ![](/images/2013-09-06-deployment-email.md1.png)

- For the trigger service select **Azure Web Sites**.

- For the trigger select **New Triggered WebJob Run**.

- For the action, we'll select **Phone** and **Call Phone** for this sample but any can be selected.

  ![](/images/2013-09-06-deployment-email.md2.png)

- Click Continue

- We need to connect to our Azure Web Site hosting our triggered WebJob, for this we need one piece of information from the Azure portal.

**This is the tricky part:**

- If your website has continuous deployment setup --> in the Azure portal go to your website, click on the **CONFIGURE** tab and under the **git** section copy the url which is under the **DEPLOYMENT TRIGGER URL**.

  ![](/images/2013-09-06-deployment-email.md3.png)

- If you don't have continuous deployment, you can author this url yourself, it is: `https://{userName}:{password}@{siteName}.scm.azurewebsites.net/deploy` where you get the `{userName}` and `{password}` from your site's publishing profile.
 
- Go back to the Zapier site and paste this url to the **Deployment URL** textbox, enter a name for this website account and click continue.

  ![](/images/2013-09-06-deployment-email.md4.png)

- Since the email action doesn't require any special account, just click continue again.

- At this point you can filter what kind of post deployment event will actually trigger this action (for example only failed deployments), for now we keep this empty, click continue

- **Create your Outbound Email** lets you customize the email, the content can be static and dynamic (coming from the post deployment result)

- On the **To** textbox enter your email

- On the **Subject** textbox enter: **Deployment complete with status: {{status}}**

- On the right side of each textbox there is an icon you can click to get the different dynamic fields that will be available from the deployment result, so in the **Body** textbox just experiment with the different fields.

  ![](/images/2013-09-06-deployment-email.md5.png)

- Continue

- **Try out your Zap** lets you test your zap by getting previous deployment results and doing the selected action on them, you can use it if you have existing deployments on your site otherwise click **skip this**.

- **Name your zap** and make it live.

- Last step is to deploy your site and see the magic (action) happens.

Get more help about [Windows Azure Web Sites on Zapier](https://zapier.com/help/windows-azure-web-sites/).
