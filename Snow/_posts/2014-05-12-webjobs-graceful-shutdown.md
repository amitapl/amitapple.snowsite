---
layout: post
title: WebJobs Graceful Shutdown
category: Azure WebJobs
---

**Azure WebJobs** are doing work and running some process you expect not to be interrupted but as in life not everything is as expected and sometimes there are interruptions which can stop your WebJob abruptly without notice aborting your process and maybe leaving your work in some bad state.

These interruptions could be due to: stopping your site, restarting your site, some configuration change to your site which causes your site to restart, Azure maintenance (version update for example) or even the VM simply crashing for some reason.

For these kind of interruptions (minus VM crash) there is a concept of a more "graceful" shutdown process for a WebJob which can help you cleanup before your WebJob is forcefully stopped.

As usual with WebJobs this concept is a bit different for continuous and triggered WebJobs, let's discuss on both.

### Continuous WebJobs ###

For continuous WebJobs Azure will notify the WebJob running process when it is about to stop it, then it'll wait a configurable amount of time (which is 5 seconds by default) after which if the process did not exit quietly it will close it.

The way Azure notifies the process it's about to be stopped is by placing (creating) a file at a path that is passed as an environment variable called `WEBJOBS_SHUTDOWN_FILE`.

Any WebJob that wants to listen on the shutdown notification will actually have to check for the presence of the file (using simple `File.Exists` function or using a `FileSystemWatcher` in whatever script language you use), when it shows up the WebJob will need to start cleaning up and break it's current loop where preferably it'll exit properly and Azure will continue the shutdown (of the site) process.

#### Here's an example using C#: ####

`
    public class Program
    {
        private static bool _running = true;
        private static string _shutdownFile;

        private static void Main(string[] args)
        {
            // Get the shutdown file path from the environment
            _shutdownFile = Environment.GetEnvironmentVariable("WEBJOBS_SHUTDOWN_FILE");

            // Setup a file system watcher on that file's directory to know when the file is created
            var fileSystemWatcher = new FileSystemWatcher(Path.GetDirectoryName(_shutdownFile));
            fileSystemWatcher.Created += OnChanged;
            fileSystemWatcher.Changed += OnChanged;
            fileSystemWatcher.NotifyFilter = NotifyFilters.CreationTime | NotifyFilters.FileName | NotifyFilters.LastWrite;
            fileSystemWatcher.IncludeSubdirectories = false;
            fileSystemWatcher.EnableRaisingEvents = true;

            // Run as long as we didn't get a shutdown notification
            while (_running)
            {
                // Here is my actual work
                Console.WriteLine("Running and waiting " + DateTime.UtcNow);
                Thread.Sleep(1000);
            }

            Console.WriteLine("Stopped " + DateTime.UtcNow);
        }

        private static void OnChanged(object sender, FileSystemEventArgs e)
        {
            if (e.FullPath.IndexOf(Path.GetFileName(_shutdownFile), StringComparison.OrdinalIgnoreCase) >= 0)
            {
                // Found the file mark this WebJob as finished
                _running = false;
            }
        }
    }

`

### Triggered WebJobs ###

For triggered WebJobs there is no shutdown notification but there is a graceful period (30 seconds by default) where the WebJob will not be forcefully shutdown immediately, the graceful period is configurable.

### Updating the graceful period ###

The graceful period can be updated for any WebJob, the way to do it is to create a file called `job.settings`  with the following content:

`
{
  "stopping_wait_time": 60
}
`

> The time is specified in seconds

This file is representing a json object of your WebJob's setting, for now the only meaningful settings are `stopping_wait_time` and `is_singleton` (for continuous WebJobs to set them to run only on a single instance).



If you have any questions on this topic feel free to leave comments.
