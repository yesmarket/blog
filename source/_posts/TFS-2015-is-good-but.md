---
title: TFS-2015 is good, but...
language: default
date: 2016-04-05 14:18:23
tags:
	- TFS
	- dev
	- TF Build
	- automation
---

We installed TFS-2015 at my organisation a few months back and I have to say; I'm starting to really like the ALM stuff. That said, there are a few things aren't quite up to my expectations and have caused me a few frustrations.

Before I get onto some of my frustrations, let me tell you about some of the good stuff.
* Team Foundation Build - It's so easy to create build definitions using pre-defined build tasks. The first team-project took about 30mins to configure the entire process, which included restoring packages from a local nuget repo, compiling the solution, running unit tests and creating an octopus deployment package.
* Tools like capacity planning etc., make the whole sprint planning process really easy to manage.
* Building charts from work-item queries and displaying them on management dashboards is so slick and easy.

There are obviously a lot of other cool things, but the list above are the ones that really impressed me.

Ok, so now onto some of the no-so-good stuff...

The first thing, which is probably the most significant is the fact that you can't include work-items from different team-projects into a single sprint. This is a major issue for my team, as we're moving to somewhat of a microservices architecture where individual team-projects represent a single distinct business capability. Individual projects will rarely focus on a single capability; most projects are for automating business process and touch multiple business capabilities. To solve this issue we had to implement a pretty significant work-around whereby we created a 'virtual' team-project as a bucket for all work-items. There were other work-arounds documented in various blogs online, but for mine this one was the least shit. Each of the various work-arounds results in a different set of issues. Some of the issues that we've encountered as a result of our chosen work-around include:
* We can't include 'code tiles' or any other code related widgets on any of our virtual team-project dashboards as the code is under a different team-project.
* When checking-in code, it is possible to reference work-items from a different team-project, however it causes a few issues with the whole code review process.

The above issue seems to be a pretty fundamental issue with TFS and a major oversight by Microsoft.

Most of the other things are all pretty minor and will *hopefully* be fixed as the product matures:
* You can apply arbitrary tags to work-items, however you can't use them as the grouping column in a chart. I would have thought this would be one of main reasons for using tags.
* There is no way to add calculated columns to your queries - think calculated fields in SSRS datasets.
* There are some very basic [variables](https://msdn.microsoft.com/en-us/library/dd286638.aspx#qvariables) that you can use in your queries e.g. @Today, however this seems to be a bit lacking. I wanted to create a query for work-items completed for the current month, but this isn't possible.
* I wanted to create some more complex reports in SSRS to display on the management dashboard, but there is no out-of-the-box SSRS reporting widget.
* Lack of templating in TF-Build.
* You can't create multiple backlog boards for an individual team-project.