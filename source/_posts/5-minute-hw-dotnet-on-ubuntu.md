---
title: 5 minute hw dotnet app on ubuntu
language: default
date: 2016-04-19 22:31:01
tags:
	- .net core 1.0
	- ubuntu
	- yeoman
---

Ok so 5 minutes might be a tad optimistic... when I say 5 minutes, I acutally mean 5 minutes from when you've got all the required apt and npm packages installed.

I'm using an Ubuntu 14.04 LTS VM for this walkthrough. No guarantees it'll work for other distributions/versions, but only one way to find out...

Anyways, first thing to do is install ASP.NET 5 (soon to be renamed to ASP.NET Core 1.0). If you run into any trouble following my instructions; use the [gettting start guide](http://docs.asp.net/en/latest/getting-started/installing-on-linux.html) as a reference.

Basically we first need to install the dotnet version manager (DNVM) so that we can then install the appropriate .NET execution environment (DNX). For our purposes we want the DNX for .NET core.

Download and install DNVM:
    

    curl -sSL https://raw.githubusercontent.com/aspnet/Home/dev/dnvminstall.sh | DNX_BRANCH=dev sh && source ~/.dnx/dnvm/dnvm.sh

Once you've installed DNVM run <code lang="sh" linenumbers="off">dnvm</code> to display it in the console:
![](/images/dnvm.png)

Next install all the DNX prerequisites:


    sudo apt-get install libunwind8 gettext libssl-dev libcurl4-openssl-dev zlib1g libicu-dev uuid-dev

Use DNVM to install DNX for .NET Core:

    
    dnvm install -r coreclr -arch x64 latest

Note; omitting the -arch will install the 32-bit version of .NET Core. If you want the 64-bit version, you must specify processor architecture.

Once you've done all this run <code lang="sh" linenumbers="off">dnvm list</code> to list all the DNX versions you've got installed on your machine. If you've got more than one, you can run <code lang="sh" linenumbers="off">dnvm use</code> to switch between different versions.

Now that we've got the DNVM and DNX for .NET Core installed we're ready to create a cross-platform Hello-World console app.

I'm going to use my good ol' friend the [yeoman](http://yeoman.io) to scaffold a .net core 1.0 console app, but for a more comprehensive walkthrough refer to [Creating a Cross-Platform Console App with DNX](http://docs.asp.net/en/latest/dnx/console.html) from the docs.

Run the following commands to first install the [aspnet generator](https://github.com/OmniSharp/generator-aspnet), then create the HelloWorld .net core 1.0 console app:


    sudo npm i -g generator-aspnet
    yo aspnet console HelloWorld

Once you install the aspnet generator, the yeoman comes along to say hello:
![](/images/yo.png)

From here we run <code lang="sh" linenumbers="off">dnu restore</code> to download all the dependencies listed in the project.json file, which adds them to your app’s packages directory. And finally we can run <code lang="sh" linenumbers="off">dnx run</code> to execute our .net code on linux:

    dnu restore
    dnx run
<div style="float:left;">
	... and badda bing badda boom
	![](/images/hello-world-output.png)
</div>
