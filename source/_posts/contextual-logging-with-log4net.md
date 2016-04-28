---
title: contextual logging with log4net
language: default
date: 2015-07-08 13:06:34
tags:
  - .NET
  - dev
  - log4net
---

If you're using a logging framework like [log4net](https://logging.apache.org/log4net/) or [nlog](http://nlog-project.org/) and you've got multiple log messages being written concurrently from different objects in the call stack, you should consider contextual logging.

One of the main benefits is that it allows you to associate and filter log messages based on a particular context. This definition is a little abstract - its probably one of those times where its easier to explain the consequences of not doing this, as opposed to the benefits of doing so... If you don't have contextual logging and you've got concurrent messages being logged, often from multiple applications and/or servers, to some centralised log store (e.g. a database table, or something like [graylog](https://www.graylog.org/)); depending on the number of log messages coming in, it could be extremely difficult to correlate log messages to a particular call context.

Consider the following scenarios:

*   A WCF service operation that gets invoked roughly 10 times per second and writes log messages from multiple different objects in the call stack.
*   A multi-threaded message queue consumer application running on multiple server instances where the consumer writes log messages at various levels in the call graph.

Now consider all the above scenarios getting logged to the same centralised logging repository!

If you were notified of an error, you might be able to easily find the error message, but without contextual logging the prior logs leading up to the error (e.g. a log of the request object or the queued message) would be very hard to reconcile. What you need is some kind of identifier (e.g. such as a GUID) associated to the call context, which you can subsequently filter on. This would then allow you to get a sequence of log messages up to and including the error from the same logical thread context.  

log4net allows you to add contextual properties to a thread properties map using the static `LogicalThreadContext.Properties` instance. The only problem with this is that it's static and not very conducive for testing - more on this later. For now, we can add context as follows:

This context can now be included in log messages by including `%property{context}` in the layout as shown below:

```xml
<appender name="TraceAppender" type="log4net.Appender.TraceAppender">
  <layout type="log4net.Layout.PatternLayout">
    <conversionPattern value="%d %-5p %property{context} %c - %m%n" />
  </layout>
</appender>
```

If we were logging to a database, we could include the context as one of the columns.

![log4net-context-logged-too-database-example](/images/log4net-context-logged-too-database-example.png)

This is especially useful when debugging, as you can locate the error log and then filter on the context of that error to get a full log trace.

The point was raised about `LogicalThreadContext.Properties` not being particularly conducive for testing. In addition it's something that should probably be abstracted out of the code as an aspect, because it's probably unrelated to any business requirements and could be thought of as boilerplate. To deal with this, I've created a [log4net.Extensions](https://github.com/yesmarket/log4net.Extensions) library that can be used to add testability to contextual logging:

```cs
public class MyService : IService
{
    private readonly ILogContextFactory _logContextFactory;

    public Service(ILogContextFactory logContextFactory)
    {
        _logContextFactory = logContextFactory;
    }

    public void DoSomething()
    {
        using (_logContextFactory.New().With("context", Guid.NewGuid()))
        {
            // ...
        }
    }
}
```

I've also created an [log4net.AutoFac](https://github.com/yesmarket/log4net.Extensions) library, which allows for abstracting out the establishment of context altogether when using the [AutoFac](http://autofac.org/) IoC container.

The log4net.Extensions and log4net.AutoFac libraries have been pushed to nuget. Be sure to check them out and let me know what you think.