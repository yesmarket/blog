---
title: SpecFlow - who writes the specifications?
tags:
  - .NET
  - Cucumber
  - dev
  - SpecFlow
  - Testing
language: default
date: 2015-08-03 10:57:41
---

My DEV team and I used [SpecFlow](http://www.specflow.org/) for our last project. It’s a great tool for acceptance test driven development (ATDD). Some of the main benefits were it a) improved the understanding of the requirements, it b) absolutely ensured our solution addressed all the requirements, and c) it improved the accountability of defects i.e. was it a coding defect or a requirements defect.

**So who writes the specifications (a.k.a. features)?**

Ideally (in my opinion), a technical BA if you’re lucky enough to have one of these, otherwise this should be a joint effort between the BAs and DEVs. The reason for this is because some level of technical nous is required to write good executable specifications using the business-readable [cucumber](https://cucumber.io/) language, however, if this task is left exclusively to the developers the acceptance-criteria will likely have some leakage of implementation concerns (i.e. how not what).