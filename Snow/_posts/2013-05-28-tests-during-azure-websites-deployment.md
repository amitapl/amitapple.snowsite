---
layout: post
title: Run tests during Azure Web Apps (Websites) deployment
category: Azure Websites, Azure Web Apps
url: /post/51576689501/testsduringazurewebsitesdeployment
---

**Deploying** websites to Azure using GIT has never been easier, and now a new support for running .NET unit tests was added.

To run tests during website deployment you'll need to customize your deployment by generating a deployment script, this is described in the following [article](/post/38418009331/azurewebsitecustomdeploymentpart2 "Microsoft Azure Web Apps - Custom Deployment Scripts Generator").

### In a nut shell: ###

* Make sure you have node.js installed on your machine.
* Open a command shell window.
* Install azure-cli (if you don't have it yet) using the command: ```npm install azure-cli -g```.

Now that prerequisites are there, go to your root repository directory in the same command shell and generate the custom deployment script.

### For an asp.net web application run the following command: ###

```azure site deploymentscript --aspWAP  -s ```

For example:

```azure site deploymentscript --aspWAP MyWebApp\MyWebApp.csproj -s MyWebApp.sln```

This will generate a ```deploy.cmd``` file under the root of your repository, we need to update this file in order to make the unit tests run during deployment.

### On the default ```deploy.cmd``` file we have 2 steps: ###

1. Build site (to a temporary path).
2. Copy changed files from the temporary path to your wwwroot.

### We'll add 2 steps in the middle like so: ###

*1. Build site (to a temporary path).*

**2. Build test project (because the first step will only build whatever is required for the site and not other projects that may exist in the repository).**

**3. Run tests.**

*4. Copy changed files from the temporary path to your wwwroot (but only do this step if tests passed/build was successful).*

### To do that add the following code between step ```:: 1.``` and step ```:: 2.``` like so: ###


    :: 1. Build to the temporary path
    %MSBUILD_PATH% "%DEPLOYMENT_SOURCE%\MvcTest\MvcTest.csproj" /nologo /verbosity:m /t:Build /t:pipelinePreDeployCopyAllFilesToOneFolder /p:_PackageTempDir="%DEPLOYMENT_TEMP%";AutoParameterizationWebConfigConnectionStrings=false;Configuration=Release /p:SolutionDir="%DEPLOYMENT_SOURCE%\.\\" %SCM_BUILD_ARGS%
    IF !ERRORLEVEL! NEQ 0 goto error

----------


    :: 2. Building test project
    echo Building test project
    "%MSBUILD_PATH%" "%DEPLOYMENT_SOURCE%\MvcTest.Tests\MvcTest.Tests.csproj"
    IF !ERRORLEVEL! NEQ 0 goto error

    :: 3. Running tests
    echo Running tests
    vstest.console.exe "%DEPLOYMENT_SOURCE%\MvcTest.Tests\bin\Debug\MvcTest.Tests.dll"
    IF !ERRORLEVEL! NEQ 0 goto error

----------

    :: 4. KuduSync
    call %KUDU_SYNC_CMD% -v 50 -f "%DEPLOYMENT_TEMP%" -t "%DEPLOYMENT_TARGET%" -n "%NEXT_MANIFEST_PATH%" -p "%PREVIOUS_MANIFEST_PATH%" -i ".git;.hg;.deployment;deploy.cmd"
    IF !ERRORLEVEL! NEQ 0 goto error


As you can see the test runner being used is ```vstest.console.exe``` which is new for VS 2012, if you have VS2012 you can run this script locally (in a VS2012 command shell) to make sure your updated script works.

```vstest.console.exe``` will recognize any test that exists in the supplied test assembly and will run it whether the test is mstest, xunit or nunit, and they could all be in the same assembly.

For a full working example of this capability try the following GitHub repository (which you can deploy on a test site just to get the feel of it): [](https://github.com/KuduApps/MvcAppWithTests/)
