---
title: Goodbye WordPress hello Hexo
language: default
date: 2016-02-27 23:36:37
tags:
	- WordPress
	- Hexo
	- GitHub
	- AWS
	- staticgen
---

About half a year ago I started this blog. I played around with a few diferent CMS platforms, but eventually settled on WordPress. At the time the main reasons I chose WordPress were because of the large number of available themes and plugins, as well as the existance of (free tier eligible) WordPress AWS AMIs. Basically, it was the quickest and easiest option based on my knowledge at the time. Since that time, my AWS free tier eligibility has expired and I haven't exactly been prolific in my blogging - something I intend to change. Anyways, I started investigating static site generators with the intent of publishing a static site to S3 so that I could shut down my WordPress EC2 instance. Evidently I didn't even need to use S3, instead opting to use GitHub pages, which is itself a discussion for another day.

When it came to choosing the static site generator platform, there were many options to choose from as per [staticgen.com](http://www.staticgen.com). I couldn't possibly trial them up w ith a few criteria to narrow the field. The criteria was as follows:
- open source
- amount of DEV activity (as evidenced by the number of GitHub stars and forks)
- language
- simplicity
- intent (i.e. whether the plaform was specifically intended for blogs)
- existing themes that I like
- plugins

From this list of criterias, I came up with [Octopress](http://www.staticgen.com/octopress) and [Hexo](http://www.staticgen.com/hexo). I gave both a quick trial run, but eventually chose Hexo because of how easy it was to use as well as the fact I found a really simple bootstrap theme that I liked.

I probably took me about an hour all up to port my WordPress blog to Hexo and to get it up and running. To show how easy it was, here are the steps that I took:

1. Install Hexo as well as the [server module](https://hexo.io/docs/server.html) for Hexo:
<code lang="posh" linenumbers="off">$ npm install -g hexo-cli</code>
<code lang="posh" linenumbers="off">$ npm install hexo-server --save</code>
2. Do the Hello World thing:
<code lang="posh" linenumbers="off">$ hexo new hello-world</code>
3. Start the Hexo server. The website will run at localhost:4000 by default:
<code lang="posh" linenumbers="off">$ hexo server</code>
4. [Find an existing theme](https://hexo.io/themes/), and then clone your chosen theme into the themes directory. I chose the [hexo-theme-bootstrap-blog](http://cgmartin.github.io/hexo-theme-bootstrap-blog/) theme.
<code lang="posh" linenumbers="off">$ cd themes</code>
<code lang="posh" linenumbers="off">$ git clone git@github.com:cgmartin/hexo-theme-bootstrap-blog.git</code>
5. Install the hexo-migrator-wordpress plugin.
<code lang="posh" linenumbers="off">$ npm install hexo-migrator-wordpress --save</code>
6. [Migrate existing posts from WordPress](https://en.support.wordpress.com/export/).
7. Run the Hexo migrator to import your WordPress posts into Hexo:
<code lang="posh" linenumbers="off">$ hexo migrate wordpress <source></code>
8. Run the site locally
<code lang="posh" linenumbers="off">$ hexo server</code>
9. Tweek the markdown of the posts to fix any formtting/display issues.
10. Initialise git repo and push Hexo site to GitHub.
<code lang="posh" linenumbers="off">$ git init</code>
<code lang="posh" linenumbers="off">$ git add --all</code>
<code lang="posh" linenumbers="off">$ git commit -m "first commit"</code>
11. Generate the static output:
<code lang="posh" linenumbers="off">$ hexo generate</code>
12. Push the generated output to a to a seperate GitHub pages repo. Note: the output should be excluded from the main Hexo repo via the .gitignore
13. Setup DNS CNAME entry to point to the GitHub pages URL.
14. Wait a couple minutes for the DNS configurations to take effect.

The only other thing to do is terminate my WordPress EC2 instance and profit!