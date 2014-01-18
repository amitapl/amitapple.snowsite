---
layout: post
title: Azure Mobile Services - Admin Table Viewer Sample
category: Windows Azure Mobile Services
url: /post/30386974705/azure-mobile-services-admin-table-viewer-sample
---

In this post I'm going to write about building a simple Windows 8 app for administration of a Windows Azure mobile service.

This app will allow us to view all of our mobile service tables with added options to delete and edit rows.

_Disclaimer: The code in my post will focus more on how to use the Windows Azure mobile services SDK to create administration apps and less on Windows 8 client code._

The first thing we want to do in an admin app is have full access to your tables, the way to do that is by using your mobile service master key as mentioned in a previous [post](/post/30386969040/azuremobileservicesadminaccess).

<script src="https://gist.github.com/3484790.js?file=AdminServiceFilter.cs"></script>


We wouldn't want to store the master key in our app's code so we actually use it as a pin for our app to let only users that know the master key actually be able to use this app.

So we're creating a master key input page that will show first.

<script src="https://gist.github.com/3484790.js?file=MasterKeyInputPage.xaml"></script>
<script src="https://gist.github.com/3484790.js?file=MasterKeyInputPage.xaml.cs"></script>


Next we want a generic way to get rows from whatever table is in our mobile service, for that we'll use the un-typed MobileServiceClient.GetTable() to get a mobile service table (IMobileServiceTable) with un-typed  operations like: ReadAsync which gets as an input the [odata](http://www.odata.org/developers/protocols/uri-conventions) query (empty string for all rows) and returns the rows as Json (IJsonValue).

<script src="https://gist.github.com/3484790.js?file=GetTableItems.cs"></script>

Except for read there are un-typed versions for update, delete and insert, those are getting a JsonObject as an input, in our sample we use delete and update, for delete we pass the JsonObject we want to delete from the Json array of rows we read, for update we pass that JsonObject with altered values by what we want to update.

Once we know all that what is left for us to do in our admin app is to create the client code that uses this, so in order to bind the Json results to our generic table (two way bind to allow editing) we need some wrapper class over the results.

<script src="https://gist.github.com/3484790.js?file=MobileServiceTableItem.cs"></script>

In our main page we'll create a dynamic table that can bind to our MobileServiceTableItems and add delete, update and refresh app bar buttons.

<script src="https://gist.github.com/3484790.js?file=MainPage.xaml"></script>

And add all other code-behind implementations.

<script src="https://gist.github.com/3484790.js?file=MainPage.xaml.cs"></script>

You can find the entire project in this [link](https://github.com/amitapl/MobileServiceAdminTableViewer).
