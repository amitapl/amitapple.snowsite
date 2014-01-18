---
layout: post
title: Azure Mobile Services - Admin Access
category: Windows Azure Mobile Services
url: /post/30386969040/azuremobileservicesadminaccess
---

So you already took the first step and created your own Windows 8 App backend in Azure.

You've added windows live authentication and have your own Users table.

You've even added a couple of server-side scripts, for example to limit a read operation on your table to get only items related to the current authenticated user.

But now you want to have some administration control over your data.

One way to do it is to create an admin Windows 8 app that uses the Azure mobile service sdk which I find currently as the simplest way.

But there is one glitch, since your permissions might be set to User Authenticated (for user related tables) you actually need to login to get that table and even if you do login, your script will only return that user's related rows only.

##The solution:

1. Use your master key (you can find it under Configure --> Manage Keys) to get administrator privileges, the master key is added as a header to the http request made to your mobile service and allows you to access any of your tables configured with any permission.

2. Add "noScript=true" as a query parameter to the http request made to your mobile service, this (must be accompanied with the master key) will disable the server-side script which allows you to handle raw table data.

_Getting to the Master Key:_

![](http://media.tumblr.com/tumblr_m9caknPGsc1rvdhx0.png)


In order to alter the http request as required by the solution we'll use the IServiceFilter interface from the Mobile Services SDK which allows us to hook up to all requests made by the SDK to our mobile service just before they are sent.

Here is the code:
<script src="https://gist.github.com/3446947.js"> </script>

And to link the filter:
<script src="https://gist.github.com/3447179.js"> </script>


**A word of caution - do not publish an app with the master key as users of your app may get to it.**
