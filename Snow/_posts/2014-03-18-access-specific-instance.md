---
layout: post
title: Request for a specific Azure website instance
category: Windows Azure Websites
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


The problem that we have now is getting this instance id, currently there is no API that will give us all the current instance ids, but a specific site can find out it's own instance id by looking at the environment variable called: **WEBSITE_INSTANCE_ID**.

So one application for this is that we can create a WebJob that is able to call the Website it is hosted on.

### WebJob Code ###

        private static void Main(string[] args)
        {
            var url = new Uri("http://siteinazure.azurewebsites.net/");
            var response = GetFromInstance(url, Environment.GetEnvironmentVariable("WEBSITE_INSTANCE_ID")).Result;
            Console.WriteLine(response.Content.ReadAsStringAsync().Result);
        }

Hope this helps.
