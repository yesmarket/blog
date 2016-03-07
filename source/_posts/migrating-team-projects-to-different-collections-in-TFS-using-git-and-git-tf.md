---
title: Migrating team projects to different collections in TFS using git and git-tf
language: default
date: 2016-03-07 21:47:17
tags:
	- git
	- GitHub
	- TFS
	- git-tf
	- PowerShell
---

When I changed jobs about a year ago, I went from using git at my previous workplace, back to TFS. About 6 months in I decided to start using git locally with TFS as the remote and [git-tf](https://gittf.codeplex.com) as the integration layer. This has worked pretty well as git enables me to be more productive in my role e.g. creating and switching between branches on a whim, logging/diffing, rebasing etc.

Recently we decided to migrate to TFS 2015. The test migration upgrade was pretty smooth, but our existing team projects were very inconsistent in terms of naming conventions, process templates, branching strategies, security settings etc., also things that were broken in the old version (e.g. all the SSRS reports) were also broken in the new version after the migration. For these reasons, we decided we would do a clean install of TFS 2015 and migrate all the source code and work items across. We investigated a few tools, but most of them were pretty flakey or didn't meet our requirements such as keeping a history of all check-ins.

Since working with git-tf I had successfully deep cloned team projects from TFS with all their history and then performed commits (checkins) back to TFS with no issues. I had the thought that I could use git and git-tf to migrate all the source code to the new version of TFS. I started playing around with this idea and found that this was indeed possible. The only thing that I couldn't do was migrate across the work item associations, but we'd already made this concession as the clean install approach meant that work items would get migrated across with new identifiers. A work item checkin policy had never really been enforced anyway and we came to the conclusion that this was a small price to pay for getting a fresh new system. As an aside, a work item check in policy will be enforced under TFS 2015.

There were roughly 50 team projects of varying size that needed to get migrated, so I decided to script the whole thing in PowerShell. This included the following:
- creation of a new team project with a default master branch in the new collection using native-TFS and TFS-power-tool commands
- deep cloning the team project from the old collection using git-tf
- deep cloning the team project from the new collection using git-tf
- merging the old into the new - all done using native git commands
- checking everything in using git-tf
- setting up long running branches and folder structure for gitflow using native TFS commands and git-tf
 
I scripted everything into a PowerShell module called TfsMigrationExtensions, which is on GitHub. I used commands from this module as follows on a per team-project basis:
 
    $ Initialize-TeamProject -collection $destinationcollection -teamprojectname $destinationname -workspace $workspacename
    $ Copy-TeamProject -sourcecollection $sourcecollection -sourcename $sourcename -sourcebranch $sourcebranch -destinationcollection $destinationcollection -destinationname $destinationname -usermappath $usermappath
    $ Set-GitFlow -collection $destinationcollection -teamprojectname $destinationname


A few things to note.

*user mapping*
There were a few times where user-mapping issues were encountered during the git-tf checkin. Presumably check-ins by someone that was removed from AD at some stage. When this happened, a USERMAP file was automatically created in the .git directory. I basically just modified the mappings, deleted the team project that was created using TFSDeleteProject command, and then re-ran the process, but this time specifying a -usermappath for the Copy-TeamProject command.

*the git merge*
There were a few issues with the merging of the source repo with the destination repo in git as there were no common ancestor commits. The destination repo had some initialisation commits i.e. one for the team project creation in TFS and one for the creation of the master branch, while the source repo had all the check-ins from the source TFS collection. To get this to work I had to try a few different things. Eventually I got this working by doing the following steps:
1. setting the old/source repo as a remote of the (destination) master
2. creating a new temp branch off this remote and switching to this branch
3. rebasing from master
4. swiching back to the master branch
5. (fast-forward) merging the temp branch into master.

Even though there were no common commits, the rebase still worked! The git commands are as follows:

    $ git remote add $source "..\$source\.git\"
    $ git fetch $localSource
    $ git checkout "remotes/$source/master" -b $source
    $ git rebase master
    $ git checkout master
    $ git merge $source