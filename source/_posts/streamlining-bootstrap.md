---
title: Streamlining Bootstrap
language: default
date: 2016-06-01 23:41:23
tags:
	- bootstrap
	- yeoman
	- gulp
	- SASS
	- Sublime Text
	- Emmet
---

At my current workplace we need to smash out responsive pages at a pretty fast rate. As far as building the responsiveness, we've decided to specialise a bit whereby we have a guy working with the marketing team to build the responsive pages. These are subsequently given to the DEVs for data binding and wiring up to the back-end. The current approach *was* to build everything in [Visual Studio](https://www.visualstudio.com/), but this seemed like overkill. Don't get me wrong; I love Visual Studio and have been using it since VS 2005, however it's a pretty heavy IDE for the task at hand.

I played around with a few bootstrap editors like [jetstrap](https://jetstrap.com/) and [pingendo](http://pingendo.com/), but we require a pretty high level of customisation and to tell you the truth the quality of the generated markup from these tools was far from perfect. So back to ol' faithful [Sublime Text](https://www.sublimetext.com/).

I've been playing around with [gulp](http://gulpjs.com/) a lot lately - on a side note gulp > grunt, but I digress... I started playing around with gulp to build an awesome workflow for smashing out responsive HTML using [bootstrap](http://getbootstrap.com/) and [SASS](http://sass-lang.com/). Some of the Gulp plugins that I used include:
* [gulp-inject](https://www.npmjs.com/package/gulp-inject) for injecting custom stylesheets and JavaScript into HTML pages.
* [wiredep](https://www.npmjs.com/package/wiredep) for wiring up bower dependencies into HTML pages.
* [gulp-sass](https://www.npmjs.com/package/gulp-sass) for automating the compilation of SASS into CSS.
* [gulp-autoprefixer](https://www.npmjs.com/package/gulp-autoprefixer) for injecting vendor prefixes into CSS.
* [nodemon](http://nodemon.io/) for monitoring changes in source code and automatically restarting [express](http://expressjs.com/).
* [browser-sync](https://www.browsersync.io/) for keeping multiple browsers & devices in sync when building websites.

All this was pretty cool stuff, especially when used in combination with Sublime plugins like [Emmet](https://packagecontrol.io/packages/Emmet), and bootstrap [3](https://packagecontrol.io/packages/Bootstrap%203%20Snippets)|[4](https://packagecontrol.io/packages/Bootstrap%204%20Snippets) snippets. But what is *really* cool was bringing the old mate [yeoman](http://yeoman.io/) along to the party.

![](/images/yeoman.png)

I'd used yeoman generators before, so I had the idea of writing my own generator for scaffolding out my streamlined bootstrap workflow. Unfortunately there's a few pre-requisites to all of this, so I'll give you a quick rundown (in Windows PowerShell). Note: I installed all this stuff over a number of years, so hopefully I didn't miss anything...

    # install chocolatey
    Set-ExecutionPolicy RemoteSigned
    iwr https://chocolatey.org/install.ps1 | iex
    # install a few chocolatey packages
    cinst git -y
    cinst poshgit -y
    cinst nodejs -y
    # install a few global npm packages
    npm i -g bower
    npm i -g gulp
    npm i -g yo
    # clone my yeoman generator
    git clone https://github.com/yesmarket/generator-bootstrap
    cd generator-bootstrap
    npm link
    # navigate to a location for builsing your streamlined bootstrap pages
    cd [path-to-streamlined-bootstrap-dir]
    # put the yeoman to work
    yo bootstrap
    # install all the bower dependencies
    bower install
    # open everything in sublime text
    subl .
    # create a new HTML page
    yo bootstrap:newpage
    # and finally run the gulp task runner to do all the things
    gulp start
    # profit :P

When you run the yo bootstrap command you'll have to answer a few questions, which influence how everything is generated:

![](/images/yops1.png)

Similarly, when you run the yeoman sub-generator yo bootstrap:newpage you'll have to tell the yeoman the name of your page, which will be generated from a template.

![](/images/yops2.png)

My yeoman generator is on [GitHub](https://github.com/yesmarket/generator-bootstrap) and free to use. Enjoy.