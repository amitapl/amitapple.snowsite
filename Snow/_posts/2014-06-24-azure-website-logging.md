---
layout: post
title: Azure Website Logging - Tips and Tools
category: Azure Websites, Logging, Azure Site Extensions
---

Using Azure Websites includes many benefits that come just out of the box, you just need to know that they're there and how to use them properly, logging is one of those benefits that integrate seamlessly to your Azure website and in this post I'll show ways on maximizing the logging experience.

## Log Types ##

These are the different log types you can get for your Azure website:

* **HTTP Logs** - Also known as iis logs, this will log all requests to your website in [W3C Extended Log File Format](http://www.microsoft.com/technet/prodtechnol/WindowsServer2003/Library/IIS/676400bc-8969-4aa7-851a-9319490a9bbb.mspx?mfr=true), the logs can be stored to your website's file system or to an Azure blob storage and can include retention (removing files older than X days).
* **Application Logs** - See in the next section.
* **Detailed Error Messages** - Detailed version of the html files produced when you website responds with an error message, this is good to enable for debugging some error responses in your website, it is stored in the website's file system.
* **Failed Request Tracing** - Also known as FREB, here you can get lots of information from IIS through it's different stacks for each failing request, read more about this [here](http://blogs.iis.net/webtopics/archive/2009/06/12/troubleshooting-a-simple-error-message-using-freb.aspx), it is also stored in the website's file system.
* **Eventlog.xml** - You may see this file sometimes under your LogFiles directory of your website (`d:\home\LogFiles`), this file contains ETW designated events, usually it is generated and populated with errors of some crash that occurred.
* **Kudu Traces** - In your website's file system under `d:\home\LogFiles\Git\trace` you can find the traces file for [Kudu](https://github.com/projectkudu/kudu) which drives some of the developer experience features of Azure Websites like: git deployment and WebJobs.

> * Log files stored in the website's file system will show up under `d:\home\LogFiles`.
> * File system log files will have some retention policy for each type:
>   * Http logs have a maximum size per log file and per sum of all log files (which is configurable in the Azure portal).
>   * Similar for application logs, they can get to a total of up to 1 MB after that old files are removed.
>   * Detailed error messages and freb have a maximum amount of files (each file consists of a single error). 


----------


![Setting different logs in the Azure portal](/images/sitediagnostics.png)

*Setting different logs in the Azure portal*


----------


## Application Logs ##

These are the logs coming from your Application/Service/Website/WebJob.

### Websites ###

If you're using ASP.NET it's simple to write application logs, just use the `Trace` class, for example:

            Trace.WriteLine("Message"); // Write a verbose message
            Trace.TraceInformation("Message"); // Write an information message
            Trace.TraceWarning("Message");
            Trace.TraceError("Message");

In the Azure portal you can direct different verbosity levels to different targets (at the same time), the targets are: file system, Azure table storage and Azure blob storage.

**For example** you can have all Information level (and up including Warning and Error) logs go to Azure table storage and all logs (including Verbose and up) go to blob storage.


----------

![Setting application logs in the Azure portal](/images/applicationdiagnostics.png)

*Setting application logs in the Azure portal*

----------

**For node.js** websites the way to write application logs is by writing to the console using `console.log('message')` and `console.error('message')` which goes to Information/Error level log entries, currently the only supported target for the log files for node.js is the file system.

Other web site types like php and python are not supported for the application logs feature.

### WebJobs ###

**Triggered** (Scheduled/On Demand)

Whatever is written to console output and console error **will go to a log file for the specific triggered webjob run**, you can see it on the WebJobs dashboard but the file itself is located under `d:\home\data\jobs\triggered\{jobname}\{jobrunid}`.

**Continuous**

Whatever is written to console output and console error **will go to the application logs** as log entries with log level Information/Error, the first 100 log entries when the continuous WebJob starts will also show up in the continuous WebJob log file also available on the WebJobs dashboard, the file itself is under  ` d:\home\data\jobs\continuous\{jobname}`.

**.NET WebJobs**
If you're using .NET you can follow the same guideline as for an ASP.NET website, once you use the `Trace` class, your traces are handled as application logs (including triggered WebJobs).

### Application Logs Fields ###

Here is the list of fields each application log entry consists of:

* **Application Name** - The website name.
* **Date Time**
* **Level** - Log level.
* **Event Id**
* **Instance Id** - A unique id for the VM running the website where the log entry came from.
* **Process Id**
* **Thread Id**
* **Activity Id** - The current (at the time of the log) activity id.
* **Message**

### Activity Id ###

The activity id field can be very powerful, it can help you correlate all log entries which came from a single request.

The easiest way to use it is to enable **Failed Request Tracing** on the Azure portal, this will have a side-effect of setting an activity id for each request your website receives and this activity id will propagate to all your application logs.

> The actual proper way to set the activity id would have been using this code in the `global.asax.cs` file:
> 
>     public class MvcApplication : System.Web.HttpApplication
>     {
>         protected void Application_BeginRequest()
>         {
>             System.Diagnostics.Trace.CorrelationManager.ActivityId = Guid.NewGuid();
>         }
>     }
>
> But since ASP.NET is doing some funky things, the activity id **may** get lost (become empty) when using async operations.

**Note** that the same activity id concept would work for .NET WebJobs, there you should use: `System.Diagnostics.Trace.CorrelationManager.ActivityId = Guid.NewGuid();` before an operation that requires an activity id.


## Log Browser Site Extension ##

One more cool feature that Azure Websites release recently is the [Azure Site Extensions](http://azure.microsoft.com/blog/2014/06/20/azure-web-sites-extensions/) which is a gallery of extensions to your Azure website that can originate from Microsoft or from the community, these site extensions can be useful utilities for your website administration and one of those site extensions is the **Log Browser**.

The **Log Browser** makes it super easy for you to access all of your Azure website logs described here and it features:

* Gives you links to the different logs that you have if you have them.
* Show logs stored in your website's file system.
* Show logs stored in your blob storage (based on the current configuration - what is configured for http logs or application logs).
* View the log files in the browser (with word highlighting capability) or download them for offline viewing.
* For application logs stored in Azure table storage it has a nice UI for showing those too.

The tool itself is very self explanatory, just install it and start using.

----------

![Install the Log Browser from the new Azure Portal](/images/logbrowserazureportal.PNG)

*Install the Log Browser from the new Azure Portal*

----------

![](/images/logbrowsermain.PNG)

*Main page*

----------

![](/images/logbrowserviewlog.PNG)

*View a log file*

----------

![](/images/logbrowsertablestorage.PNG)

*View log entries from table storage*

----------


The **Log Browser** site extension is open source and is hosted on [GitHub](https://github.com/amitapl/ThatLogExtension), you can use this repository to help you get started on your own site extension idea or to contribute to the **Log Browser** site extension.


## Final Thoughts ##

Azure Websites has a very nice and powerful logging experience, together with the **Log Browser** you get an online dashboard and log viewing experience for free and with minimal effort.
