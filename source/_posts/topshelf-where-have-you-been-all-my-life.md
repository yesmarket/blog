---
title: Topshelf; where have you been all my life?
language: default
date: 2016-04-28 13:29:50
tags:
	- Topshelf
	- dev
	- .NET
	- windows-services
---

I often find out about some new (and in some cases not-so-new) tech/framework that I wish I'd known about many years beforehand. [Topshelf](http://docs.topshelf-project.com/en/latest/) is one of the most recent additions to this long list, joining old names such as [NHibernate](http://nhibernate.info/) and [AutoFixture](https://github.com/AutoFixture) as well as other more recent additions like [AWS Lambda](http://docs.aws.amazon.com/lambda/latest/dg/welcome.html) and [Masstransit](http://docs.masstransit-project.com/en/latest/)... I guess you don't know what you don't know right - One things for sure; it forms a strong case for attending user group meetings and using online training resources like pluralsight.

Anyway back to Topshelf.

If you've ever written windows services you've probably had the pain of having to deal with InstallUtil (and InstallUtil -u) on your local machine and having to run Visual Studio as administrator so that you can attach to a windows-service process. In the case of the former; sure you could script this into a post build task or something, but as a wise woman once said "aint nobody got time for dat".

<img style="float: left;" src="/images/aint-nobody-got-time-for-dat.jpg">

<div style=" clear: both;">
	Enter Topshelf.

	Topshelf allows you to build services as standard console-apps. Not only does this give you all the nice debugging features when running under Visual Studio, but you can also install and configure these 'Topshelf services' into windows via the command line. Effectively you can run your services as command line apps locally and install them as windows-services in shared environments - creating windows-service classes and using InstallUtil no more!

	Ok, so how do you set up a Topshelf service?
	Topshelf, which is available as a [nuget package](https://www.nuget.org/packages/Topshelf/), provides a fluent interface that makes it really quite simple. Basically you call the static <code lang="sh" linenumbers="off">HostFactory.Run</code> command passing in a callback parameter where you configure your service behaviour. I'm not going to go into any depth with regards to the different configuration options, suffice to say; you can pretty much do anything that you can do manually from the Windows Services console.

	Here is a simple example:

```cs
using Topshelf;

namespace Console.Host
{
    public class Program
    {
        public static void Main(string[] args)
        {
            HostFactory.Run(cfg =>
            {
                cfg.Service<IServiceManager>(instance =>
                {
                    instance.ConstructUsing(() => new ServiceManager());
                    instance.WhenStarted(manager => manager.Start());
                    instance.WhenStopped(manager => manager.Stop());
                });

                cfg.SetServiceName("TestService");
                cfg.SetDisplayName("Honourable Test Service");
                cfg.SetDescription("This is just a test");

                cfg.StartAutomatically();
            });
        }
    }

    public interface IServiceManager
    {
        void Start();
        void Stop();
    }

    public class ServiceManager : IServiceManager
    {
        public void Start()
        {
			//Start something
        }

        public void Stop()
        {
            //Stop something
        }
    }
}
```
I've personally used this approach for hosting WCF services as windows-services where my services expose net.tcp endpoints so that I can do distrubuted transactions - works like a charm.

The only other thing to mention is how to install the service into windows (in shared environments). Basically this is as simple as <code lang="sh" linenumbers="off">./NameOfConsoleApp.exe install</code>. 

For more information about what you can do via the command line run <code lang="sh" linenumbers="off">./NameOfConsoleApp.exe help</code>.
</div>
