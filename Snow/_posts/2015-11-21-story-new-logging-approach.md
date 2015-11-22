---
layout: post
title: New approach to logging - Story
category: Logging, Analytics, Telemetry, Web Applications
---

Writing logs is usually pretty easy in a project, just select/add your favorite logging framework (Trace, NLog, Log4Net, ...), maybe configure it a bit, and start writing logs.

But then comes the time when you actually need to use these logs, maybe to debug an issue, this is when you realize the mess: here is an error log but wait what happened before that, who was the user, what was the action and how long did it take... someone PLEASE put me in context!

Well there are current solutions for that, you'll go back to the code and start adding correlation ids, and maybe use some logging query magic with grouping and filtering on top of what might be a high amount of logs. Not so easy after all, especially after the fact.

In this blog post I wanted to show a new approach to logging using a new framework called: [Story](https://github.com/narratr/story).

## Story ##

    Storytelling.Factory.StartNew("MyAction", () =>
    {
        this.counter++;
        Storytelling.Info("added 1 to counter");
        Storytelling.Data["counter"] = this.counter;
    });

The story framework (currently only for .NET) is about collecting information (logs and any bits of data) about the currently running code and putting it inside a context and when this context ends using rules figure out what to do with this information collection (story).

Too abstract, lets use an example:

We have a web application for posting questions, in this approach every request will run under a new story context. This means that every log produced will be added to that story and every data will be added to it like the current user posting the question, the url of the request, the response and even the query we ran to insert the question to the database.

Then when the request ends and the story stops, a set of rules will decide what to do with it:

* If there was an error store the story.
* If the operation ran too long increase a performance counter.
* If the user is *someone you like* alert the team.
* If a new question was added send an analytics event to [keen.io](https://keen.io).
* If nothing special happened send to trace (or maybe do nothing with it).

This gives you full control on your logs/telemetry/analytics at coding time, and the story you get, if there was an error for example, will have all the information you need, and more importantly, information (stories) that you don't need (usually most of them) will not distract you.

**How cool is that?**

Well these rules can also be updated on the fly which opens awesome scenarios, for example - your user has an issue, just add a rule that sends all his stories to a new storage container and observe that container, no need to filter and you see everything that this user do immediately.

## Using Story ##

To use the Story you need to create/start a new story and run your code within the context of that Story.

    Storytelling.Factory.StartNew("MyAction", () =>
    {
        // ... your code
        // and adding logs and information to the story
        Storytelling.Info("added 1 to i");  // log
        Storytelling.Data["i"] = i;         // data
    });

The created story is added to the context (CallContext or HttpContext depending on where you run) so anything running within that context can just call `Storytelling` and add logs/data to the story.

You can also create new stories, which will be added as children to the current story in context (and will become the current story in the context).

The second part is setting the rules which tells each story what to do when it ends (and begins), for example:

    // Create the ruleset
    var ruleset = new Ruleset<IStory, IStoryHandler>();

    // Add a new predicate rule, for any story run the console handler which prints the story to the console
    var consoleHandler = new ConsoleHandler("PrintToConsole", StoryFormatters.GetBasicStoryFormatter(LogSeverity.Debug));
    ruleset.Rules.Add(
        new PredicateRule(
            story => true,
            story => consoleHandler));

    // Set a new basic factory that uses the ruleset as the default story factory
    Storytelling.Factory = new BasicStoryFactory(ruleset);

> You should initialize the factory once at the start of the app.

## Adding Story to Web Application ##

To get a better real sample I'll show how to add stories to an asp.net owin web application.

> You can find the code demonstrated in this blog post in [this github repository](https://github.com/amitapl/FooWebApplication).

We'll start with a barebone owin web application and add the [Story nuget package](https://nuget.org/packages/Story) (`nuget install story`).

Then add a middleware that puts requests in a Story context.

    public class StoryMiddleware : OwinMiddleware
    {
        public StoryMiddleware(OwinMiddleware next)
            : base(next)
        {
        }

        public override Task Invoke(IOwinContext context)
        {
            var request = context.Request;
            return Storytelling.StartNewAsync("Request", async () =>
            {
                try
                {
                    Storytelling.Data["RequestUrl"] = request.Uri.ToString();
                    Storytelling.Data["RequestMethod"] = request.Method;
                    Storytelling.Data["UserIp"] = request.RemoteIpAddress;
                    Storytelling.Data["UserAgent"] = request.Headers.Get("User-Agent");
                    Storytelling.Data["Referer"] = request.Headers.Get("Referer");

                    await Next.Invoke(context);

                    Storytelling.Data["Response"] = context.Response.StatusCode;
                }
                catch (Exception e)
                {
                    var m = e.Message;
                    Storytelling.Error(m);
                    throw;
                }
            });
        }
    }

To make sure all exceptions are handled properly (and not get lost in web api), we'll add an exception filter too.

    public class ExceptionStoryFilterAttribute : ExceptionFilterAttribute
    {
        public override void OnException(HttpActionExecutedContext context)
        {
            Storytelling.Warn("Internal error - " + context.Exception);
            var message = context.Exception.Message;
            var httpStatusCode = HttpStatusCode.InternalServerError;

            Storytelling.Data["responseMessage"] = message;

            var resp = new HttpResponseMessage()
            {
                Content = new StringContent(message)
            };

            resp.StatusCode = httpStatusCode;

            context.Response = resp;

            base.OnException(context);
        }
    }

Add them to owin and web api.

    public void Configuration(IAppBuilder appBuilder)
    {
        // ...
        appBuilder.Use<StoryMiddleware>();
        // ...
        HttpConfiguration config = new HttpConfiguration();
        config.Filters.Add(new ExceptionStoryFilterAttribute());
        appBuilder.UseWebApi(config);
        // ...
    }

Now, we can start using Story to collect information.

    public Task<HttpResponseMessage> GetSomething()
    {
        // We wrap a controller action with a (sub)story
        return Storytelling.Factory.StartNewAsync("GetUser", async () =>
        {
            var name = await fooService.GetRandomName();

            // Log to the story
            Storytelling.Info("Prepare something object");
            object something = new
            {
                Name = name
            };

            // Add data to story
            Storytelling.Data["something"] = something;

            return Request.CreateResponse(something);
        });
    }

We can also collect information in inner methods.

    public async Task<string> GetRandomName()
    {
        Storytelling.Info("Getting random name");
        await Task.Delay(10);
        return GetRandomItem(Adjectives) + " " + GetRandomItem(Animals);
    }

Finally we need to initialize the story factory, we will use `FileBasedStoryFactory` which lets us update the ruleset on the fly. The ruleset itself is a class written in a file, to change it we change the code in that file and copy it over to the server.

> The ruleset code file is stored in the project both as a source file (Compile) and as a content file (marked as "Copy Always") so it has the benefit of intellisense and used as an external file to the project.

    // Startup.cs
    public void Configuration(IAppBuilder appBuilder)
    {
        Storytelling.Factory = new FileBasedStoryFactory(ConfigurationManager.AppSettings["StoryRulesetPath"]);
        // ...
    }

    // StoryRuleset.cs
    public class StoryRuleset : Ruleset<IStory, IStoryHandler>
    {
        public StoryRuleset()
        {
            IStoryHandler storyHandler =
                StoryHandlers.DefaultTraceHandler.Compose(
                new AzureTableStorageHandler("AzureTable", azureTableStorageConfiguration));

            Rules.Add(
                new PredicateRule(
                    story => story.IsRoot(),
                    story => storyHandler));
        }
    }

You'll notice we've added both a story handler that sends stories to the Trace and one that sends stories to the azure table storage.
To make the second one work you'll need to add a connection string for your azure storage account with the name "StoryTableStorage" (can be in the web/app.config or the azure portal settings if using azure).

> If you use the azure table storage story handler to persist your stories and also use Azure Web Apps to deploy your web application you can install the [Azure Websites Log Browser](https://www.siteextensions.net/packages/websitelogs/) ([blog post here](http://blog.amitapple.com/post/2014/06/azure-website-logging/)) which now supports viewing your stories in a readable way.

You can find the full sample app [here on github](https://github.com/amitapl/FooWebApplication).

It also has a **Deploy to Azure** button if you want to try it out on the cloud and try out the log browser site extension, just make sure to set the azure storage connection string (when asked). To access the log browser, go to: `https://{sitename}.scm.azurewebsites.net/websitelogs/`.

You can find more information about the Story framework [here](https://github.com/narratr/story).
