---
layout: post
title: My First Windows Store App - חדשות  8
category: Windows 8
url: /post/33776906129/myfirstwindowsstoreapp
---

Just published my first windows 8 app to the store, it is a news reader in Hebrew.

[My App On The Windows Store]( http://apps.microsoft.com/webpdp/en-US/app/8/feb11719-8b28-407a-ba6c-bcf14901e643 "חדשות 8")

![](http://media.tumblr.com/tumblr_mc1p3w6VAu1rvdhx0.jpg)

Basically it is an RSS feed reader for a specific set of feeds from different Israeli news sites.
To use it you currently have to login using your Microsoft account as it will save the current feeds you selected to follow on the main screen of the app.

Writing a Windows 8 app turns out to be quite a process, I really like the Windows SDK and tools for writing an app and it takes a fairly small amount of time to get the basic scenario going but finishing up the app took much longer than I expected.

Among things I had to do (which is a good starting check-list for writing apps for the store):
- Make sure all pages work in all layouts (including: portrait and snapped especially) and all resolutions (at least the lowest ones), using the LayoutAwarePage class you can update the way a page looks in the different layouts.
- Have all graphics ready, there are several images you have to produce like: wide tile, square tile, different Windows store images and more.
- Test your app, it is a given but sometimes when we are the dev and tester we tend to give ourselves some slack
- Create a privacy policy, there are some free tools online you can use.
- Put a link for the privacy policy in the Settings charm, I failed on this thing twice simply because I didn’t know that the settings charm is customizable per app (who figured? ), it is fairly simple.

Most of the time I spent on this app was for polishing up the graphics and understanding some XAML related quirk like finding a proper (or any in fact) way to bind the selected values of a grid view (two way bind).

On the backend for this app I used [Windows Azure Mobile Services](http://www.windowsazure.com/en-us/develop/mobile/ "") which hands down makes backend development a breeze, I use it to authenticate users (with the Live SDK also) and store their selected feeds and also to store all of the feeds the app has so I can add more feeds without updating through the store.

I’ve also authored a simple website to hold my About and Privacy Policy html pages, for that I used Windows Azure Websites, setting up a site took about a minute and that includes creating it (naming it) and setting up a cool git repository on it so deployment is as fast and easy as “git push”, this was also an awesome experience ([git deployment]( http://www.windowsazure.com/en-us/develop/net/common-tasks/publishing-with-git/ "")).

[My Website]( http://newsinhebrew8.azurewebsites.net/ "חדשות 8")

So [there](http://apps.microsoft.com/webpdp/en-US/app/8/feb11719-8b28-407a-ba6c-bcf14901e643 "חדשות 8") it is, my first Windows store app, sorry just for Hebrew speakers, soon I’ll have another one out which has more integration with Mobile Services (in certification process now).

Last note – Live Tiles are cool!!! 
