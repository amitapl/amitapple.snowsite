---
layout: post
category: Windows Azure Websites
title: Simple terminal for your Azure Website
url: /post/45675601255/azurewebsiteterminal
---

This guide will help you run server-side commands on your Azure Website hosting environment, it is very simple and useful and since it's implemented in node.js, it'll work on any (node) supporting OS (Windows / MAC / Linux).


- First, you should install [node.js](http://nodejs.org/) if you don't already have it (#whynot?).

- Install [KuduExec](http://github.com/projectkudu/KuduExec):

    ```npm install kuduexec -g```

    **Note:** There's also a .NET version of kuduexec called [KuduExec.NET](https://github.com/projectkudu/KuduExec.NET)

- Find your "kuduexec" Azure Website endpoint:

 - Add "scm" after your site's name (if you have a custom domain you still need to add this to the original URL you received from Azure), for example:
   
         http://somesitename.azurewebsites.net/ --&gt; http://somesitename.```scm```.azurewebsites.net/

 - If you have "git deployment" enabled on your site you can get the endpoint (including user name and password) from the Azure portal go to you site, under the *CONFIGURE* tab, on the *git* section in the *DEPLOYMENT TRIGGER URL*:

        ![](http://media.tumblr.com/8198cda0808027dc4c497af2a2dab92e/tumblr_inline_mjujt54Sf01qz4rgp.png)

- Now that you have your endpoint, open a shell window and run the command:

    ```kuduexec <Your endpoint URL / git deployment url>```

    For example:

    ```kuduexec http://somesitename.scm.azurewebsites.net/```

    **Note:** You can also add your user name and password to the url (otherwise it simply asks you for them):

    ```kuduexec http://username@somesitename.scm.azurewebsites.net/```

- At this point you'll see a command prompt with the root directory of your site (not "wwwroot"), some info on the [directory structure](https://github.com/projectkudu/kudu/wiki/File-structure-on-azure).

- Now you can run your commands, including shell commands such as "cd", "dir" and "copy", you'll also be able to run executables such as "git" and "npm".

    ![](http://media.tumblr.com/4ed73405533466c160ba3b8813410293/tumblr_inline_mjtbgdkD031qz4rgp.png)

- To quit simply type ```exit```

## Usage Examples

- One useful scenario this can help you with is when you want to run a **garbage collection** on your git repository, which can reduce your storage usage:
  - Simply go to the repository directory: ```cd site\repository```

  - And run: ```git gc```

- Another will be to check if you some lingering processes using: ```ps -W```

 - Here I have a lingering node.exe process and I kill it using: ```kill```

			C:\DWASFiles\Sites\somesitename\VirtualDirectory0> ps -W
			PID    PPID    PGID     WINPID  TTY  UID    STIME COMMAND
			    14620       0       0      14620    ?    0 08:26:30 D:\Windows\SysWOW64\inetsrv\w3wp.exe
			      824       0       0        824    ?    0 08:40:04 D:\Windows\SysWOW64\cmd.exe
			    28484       0       0      28484    ?    0 08:40:04 D:\Program Files (x86)\nodejs\node.exe
			    11584       1   11584      11584    ?  500 08:40:48 /bin/ps
			C:\DWASFiles\Sites\somesitename\VirtualDirectory0> kill -f 28484
			C:\DWASFiles\Sites\somesitename\VirtualDirectory0> ps -W
			PID    PPID    PGID     WINPID  TTY  UID    STIME COMMAND
			    14620       0       0      14620    ?    0 08:26:30 D:\Windows\SysWOW64\inetsrv\w3wp.exe
			    29008       0       0      29008    ?    0 08:41:08 D:\Windows\SysWOW64\cmd.exe
			    19756       1   19756      19756    ?  500 08:41:08 /bin/ps
			C:\DWASFiles\Sites\somesitename\VirtualDirectory0>



## Important notes

- This simple terminal is not for running interactive commands, running a command that requires user input will hang (for 3 minutes until recognized as hanging process and then it will be aborted) since there is no way, currently, to provide input for the running command.

- The output for a single command will arrive only after the command finish running, so if you run a long running command it'll take time for the output to show
(other than piping: ```requireInput.exe < input.txt```).
