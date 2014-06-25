---
layout: post
title: Request for a specific Azure website instance
category: Azure Websites
---

In Windows Azure Websites you have the ability to scale your site by adding more instances to it where each instance is running on a different VM.

When you have more than one instance a request made to your site can go to any of them using a load-balancer that will decide which instance to route the request to based on how busy each instance is at the time.

One feature of this load-balancer is that once a request from your browser is made to the site, it will add a "cookie" to it (with the response) containing the specific instance id that will make the next request from this browser go to the same instance.

We can use this feature to send a request to a specific instance of our site.

The name of the cookie we're going to use is: **ARRAffinity**

### Code ###

        private static async Task<HttpResponseMessage> GetFromInstance(Uri url, string instanceId)
        {
            var cookieContainer = new CookieContainer();
            using (var handler = new HttpClientHandler() { CookieContainer = cookieContainer })
            {
                using (var httpClient = new HttpClient(handler))
                {
                    cookieContainer.Add(url, new Cookie("ARRAffinity", instanceId));
                    return await httpClient.GetAsync(url);
                }
            }
        }


The problem that we have now is getting this instance id, in the update below I'll show how, but a specific site can find out it's own instance id by looking at the environment variable called: **WEBSITE\_INSTANCE\_ID**.

So one application for this is that we can create a WebJob that is able to call the Website it is hosted on.

### WebJob Code ###

        private static void Main(string[] args)
        {
            string instanceId = Environment.GetEnvironmentVariable("WEBSITE_INSTANCE_ID");
            string siteName = Environment.GetEnvironmentVariable("WEBSITE_SITE_NAME");
            var url = new Uri("http://" + siteName + ".azurewebsites.net/");
            var response = GetFromInstance(url, instanceId).Result;
            Console.WriteLine(response.Content.ReadAsStringAsync().Result);
        }


## Update ##

Azure Web Sites now provides an API to get all instances (IDs) for your website, you can either do it programmatically or using the Azure CLI tools.

### Get instance IDs for a web site - sample code ###

First thing to do is install the Azure Websites Management Library from [nuget](http://www.nuget.org/packages/Microsoft.WindowsAzure.Management.WebSites/ "nuget"), this is the SDK for managing your Azure Web Site from code.

Now all you need is this code:

    internal class Program
    {
        private static void Main(string[] args)
        {
            var cert = new X509Certificate2();
            cert.Import(Convert.FromBase64String("MIIJ/...=="));
            var client = new WebSiteManagementClient(new CertificateCloudCredentials("subscription_id_guid", cert));

            var instanceIds = client.WebSites.GetInstanceIds("westuswebspace" /*webspace name*/, "somesite" /*web site name*/);
            Console.WriteLine(String.Join(", ", instanceIds));
        }
    }


### Azure CLI tools ###

Azure has CLI tools for both PowerShell (for windows users) and xplat using node.js under the cover (for all users including mac, unix and windows).

To get these tools you can go to this [link](http://azure.microsoft.com/en-us/downloads/).

To install the xplat tool you can simply write the following command: `npm install azure-cli -g`

For more information on using the CLI tools you can go to these links:
[Managing the Cloud from the Command Line](http://www.hanselman.com/blog/ManagingTheCloudFromTheCommandLine.aspx) and [Azure PowerShell - MSDN](http://msdn.microsoft.com/en-us/library/azure/jj156055.aspx)

> Tip for PowerShell - Start by using the following command: `Add-AzureAccout`.

In both tools you get the website's instance ids by getting/showing the website.

#### PowerShell ####


    Get-AzureWebsite sitename

    Instances                   : {6d016e86bc41ff8e2fcf5d66da0116e929b41609a8cace17b40b6c5e4eb15b44}
    NumberOfWorkers             : 1
    ...


#### xPlat ####


    > azure site show sitename

    info:    Executing command site show
    info:    Showing details for site
    + Getting site information
    + Getting site config information
    + Getting repository settings
    + Getting diagnostic settings
    + Getting site instances information
    + Getting locations
    data:
    data:    Web Site Name:  sitename
    data:    Site Mode:      Standard
    data:    Enabled:        true
    data:    Availability:   Normal
    data:    Last Modified:  Mon Jun 16 2014 18:46:58 GMT-0700 (Pacific Daylight Time)
    data:    Location:       West US
    data:
    data:    Host Name
    data:    ------------------------
    data:    sitename.azurewebsites.net
    data:
    data:    Instance Id
    data:    ----------------------------------------------------------------
    data:    6d016e86bc41ff8e2fcf5d66da0116e929b41609a8cace17b40b6c5e4eb15b44
    ...



Hope this helps.
