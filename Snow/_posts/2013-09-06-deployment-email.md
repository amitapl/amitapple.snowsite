---
layout: post
title: Sending an email when your Azure web site deployment completes
category: Windows Azure Websites
url: /post/56390805814/deployment-email
---

Windows Azure Web Sites have a new support for web hooks, currently the only event that is directly being invoked is called **PostDeployment** and it will be invoked whenever a deployment is complete with the result of that deployment.

You can find more information on the API for this feature [here](https://github.com/projectkudu/kudu/wiki/Web-hooks "Kudu Web Hooks").

### Cool Stuff with Web Hooks ###

Having this new API allowed us to collaborate with [zapier.com](http://zapier.com) and allow developers to get this post deployment information in many ways like: email, phone call or sms or even tweet about it.

### About Zapier ###

[zapier](http://zapier.com) is a service that is all about the integration of other different services through the concept of **triggers** and **actions**.

A trigger could be a new github issue was opened or a new email was sent to your gmail account and an action could be tweeting a new tweet to tweeter or create a new note in your Evernote account.

In our case we've created a new trigger on Zapier for a deployment to a website that is complete and using Zapier you can connect this trigger to any of actions that are available.

More on [Windows Azure Web Sites - Zapier Service](https://zapier.com/zapbook/windows-azure-web-sites/)

**Let's see how:**

### Prerequisites ###

- A Windows Azure Website that is deployed using source control (git, mercurial or dropbox).

- Sign up to [Zapier](https://zapier.com/app/signup)

- Have both the Zapier and Azure portal open

### Steps ###

- Go to Zapier and create a new zap.

  ![](http://media.tumblr.com/bae6d050358c1a04257ba17bbab89682/tumblr_inline_mqh23m5Pko1qz4rgp.png)

- For the trigger service select **Windows Azure Websites**

- For the trigger select **New Website Deployment**

- For the action, we'll select **email** and **send outbound email** for this sample but any can be selected

  ![](http://media.tumblr.com/d3dfa3e75d8fdf2ca07b083b9ba09a6f/tumblr_inline_mso7keWNBA1qz4rgp.png)

- Click Continue

- We need to connect to our Azure Website, for this we need one piece of information from the Azure portal

- In the Azure portal go to your website, click on the **CONFIGURE** tab and under the **git** section copy the url which is under the **DEPLOYMENT TRIGGER URL**

  ![](http://media.tumblr.com/7bcf1cb2e7758cc8dbba6a7942d034aa/tumblr_inline_mso7l28Xgo1qz4rgp.png)

- Go back to the Zapier site and paste this url to the **Deployment URL** textbox, enter a name for this website account and click continue.

  ![](http://media.tumblr.com/9d634a10ff3776ded76c9f97c027a369/tumblr_inline_mso7lhv0qv1qz4rgp.png)

- Since the email action doesn't require any special account, just click continue again.

- At this point you can filter what kind of post deployment event will actually trigger this action (for example only failed deployments), for now we keep this empty, click continue

- **Create your Outbound Email** lets you customize the email, the content can be static and dynamic (coming from the post deployment result)

- On the **To** textbox enter your email

- On the **Subject** textbox enter: **Deployment complete with status: {{status}}**

- On the right side of each textbox there is an icon you can click to get the different dynamic fields that will be available from the deployment result, so in the **Body** textbox just experiment with the different fields.

  ![](http://media.tumblr.com/a21dd884c31dd46d0f67f64ae24d9108/tumblr_inline_mso7ltggoF1qz4rgp.png)

- Continue

- **Try out your Zap** lets you test your zap by getting previous deployment results and doing the selected action on them, you can use it if you have existing deployments on your site otherwise click **skip this**.

- **Name your zap** and make it live.

- Last step is to deploy your site and see the magic (action) happens.

Get more help about [Windows Azure Web Sites on Zapier](https://zapier.com/help/windows-azure-web-sites/).
