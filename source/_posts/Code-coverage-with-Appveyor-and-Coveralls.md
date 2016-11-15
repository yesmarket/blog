---
title: Code coverage with Appveyor and Coveralls
language: default
date: 2016-11-16 12:51:01
tags:
  - Appveyor
  - Travis CI
  - Coveralls
  - shields.io
  - OpenCover
  - xunit
  - nunit
  - GitHub
---

I was using [Travis CI](https://travis-ci.org) for an open source .NET library and I had just shielded the repo with a build shield for Travis. I had a quick look on [shields.io](http://shields.io) for other shields that might be appropriate for my repo and I noticed quite a few that were focussed on code coverage. Since I'd been dilligent with my unit testing, I decided it would be a good idea to add a coverage shield to my repo, I mean 'why not' right? I'll give you a reason; generating code coverage reports with Mono and [monocov](https://github.com/mono/monocov) on Travis, which is a Linux based build system, turned out to be a massive pain in the arse. Don't get me wrong there seems to be [evidence of success](http://blog.csmac.nz/monocov-travis-ci-winning/), however I lost patience pretty quickly and decided to pivot. Also the fact that monocov hasn't been maintained since 2011 was a bit of a red flag.

![](/images/different-approach.jpg)

I'd read and heard about [Appveyor](https://www.appveyor.com) - a CI/build system similar to Travis that uses Windows instead of Linux. After a bit of googling, I had a plan; Appveyor, [OpenCover](https://github.com/OpenCover/opencover) & [Coveralls](https://coveralls.io). 

The following steps got me over the line
* Created an Appveyor account from my GitHub credentials.
* Added my GitHub repo as a project in Appveyor.
* Created a Coveralls account from my GitHub credentials.
* Added my GitHub repo to Coveralls whilst taking note of the repo token (to be used for Coveralls API authorisation from Appveyor).
* Added [OpenCover](https://www.nuget.org/packages/opencover), [coveralls.net](https://www.nuget.org/packages/coveralls.net) & [xunit.runner.console](https://www.nuget.org/packages/xunit.runner.console) nuget packages to my test Visual Studio project, which was using the [xunit](https://xunit.github.io) test framework.
* Securified the Coveralls repo token so this information is never public.
* Added [appveyor.yml](https://www.appveyor.com/docs/appveyor-yml) to my GitHub repo (the core bits are shown below):
* Committed and pushed the repo.

```yml
version: '1.0.{build}'
skip_tags: true
configuration: Release
assembly_info:
  patch: true
  file: '**\AssemblyInfo.cs'
  assembly_version: '{version}'
  assembly_file_version: '{version}'
  assembly_informational_version: '{version}'
environment:
  COVERALLS_REPO_TOKEN:
    secure: fKaSS6YrHCgMckGG62aj1A4WnXYeUnps/SQe8RXpK//ZDl8bCtH3O37Ar/TxMvs0
before_build:
  - nuget restore ".\Foo.sln"
build:
  project: .\Foo.sln
  verbosity: minimal
test:
  assemblies:
    - Foo.UnitTests
after_test:
  - ps: >-
	mkdir coverage-results
	.\packages\OpenCover.4.6.519\tools\OpenCover.Console.exe -register:user -target:.\packages\xunit.runner.console.2.1.0\tools\xunit.console.exe "-targetargs:.\Foo.Tests\bin\Release\Foo.Tests.dll -noshadow" "-filter:+[Foo*]* -[*Tests]*" -skipautoprops -output:.\coverage-results\results.xml
	.\packages\coveralls.net.0.7.0\tools\csmacnz.Coveralls.exe --opencover --input .\coverage-results\results.xml --repoToken $env:COVERALLS_REPO_TOKEN --commitId $env:APPVEYOR_REPO_COMMIT --commitBranch $env:APPVEYOR_REPO_BRANCH --commitAuthor $env:APPVEYOR_REPO_COMMIT_AUTHOR --commitEmail $env:APPVEYOR_REPO_COMMIT_AUTHOR_EMAIL --commitMessage $env:APPVEYOR_REPO_COMMIT_MESSAGE --jobId $env:APPVEYOR_JOB_ID
```

Now I'm not going to claim this worked first time! I had to experiment with the OpenCover console quite a bit (with [ReportGenerator](https://github.com/danielpalme/ReportGenerator)) to get everything working. But once it *was* all working, it was pretty neat.

The only thing left to do was shield my repo:

![](/images/shields.png)

oh and I also got OpenCover working with [nunit](https://www.nunit.org):

    .\packages\OpenCover.4.6.519\tools\OpenCover.Console.exe -register:user -target:.\packages\NUnit.ConsoleRunner.3.5.0\tools\nunit3-console.exe -targetargs:.\Foo.Tests\bin\Release\Foo.Tests.dll "-filter:+[Foo*]* -[*Tests]*" -skipautoprops -output:.\coverage-results\results.xml
