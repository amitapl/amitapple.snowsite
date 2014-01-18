---
layout: post
title: Custom Parameters in Azure Mobile Services
category: Windows Azure Mobile Services
url: /post/30921523746/customparametersinmobileservices
---

Windows Azure Mobile Services is using SQL Azure as the backend, one of the benefits you have with a SQL database is the ability to join tables simply, but currently that ability is not manifested in an easy way through the server-side scripts or the client SDK of the mobile services.

So when I needed to get rows by joining 2 tables, I used this following simple solution.

In my case I had 2 tables, one for users with the user id and user name among other columns, and the other for user groups with information on which user is a part of which group, there I had the same user id with other group related information.

I needed a view with all of the user group columns together with the name of the user.

To do that I've created a new table called **UserGroupsExtended** and I've set permissions to admin for all operations except read, this is a workaround for creating a new endpoint in our mobile service.

![](http://media.tumblr.com/tumblr_mb772oiGgk1rvdhx0.jpg)

In the read operation server-side script of this new table, I have the following code in which I use a SQL query to join the 2 tables and return the results of that query (instead of returning results related to the original UserGroupsExtended table which will always be empty).

<script src="https://gist.github.com/3809541.js?file=UserGroupExtended.read.js"></script>

In this script I actually do more than that, I use a custom parameter (as described [here](/post/30921523746/customparametersinmobileservices "custom parameters in mobile services")) to pass the specific group id I want to get results on, this is since the query object is for the original UserGroupsExtended table which we are not really using.

And I also added this "in" part to only get results for groups where the current user is part of.

On the client side I've added a new UserGroupExtended class which derives from UserGroup and adds the user name as a property.

<script src="https://gist.github.com/3809541.js?file=UserGroupExtended.cs"></script>

And the last piece is for actually getting a result back from this new view.

<script src="https://gist.github.com/3809541.js?file=Data.cs"></script>

The last piece assumes you have the client-side code mentioned on the [custom parameters post](/post/30921523746/customparametersinmobileservices "custom parameters in mobile services").
