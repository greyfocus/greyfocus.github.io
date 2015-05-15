---
layout: post
title:  "Running Jekyll on Windows using Vagrant"
date:   2015-05-03 00:10:19
categories: jekyll
keywords: jekyll, vagrant, windows, static site generator 
---

After reading a bit about static site generators I've decided to give them a
try. One of the most popular choices is Jekyll, which is also the engine behind
GitHub Pages. 

<!-- more -->

One of the main disadvantages of static site generators is that they require a
minimal setup on the local machine, in order to edit posts.  Jekyll is based on
Ruby and due to its dependencies, it's not officially supported on Windows.
There are some success stories though, but I didn't want to have a complicated
setup, especially since I may want to replicate it on other computers. 

## Setup
The following steps describe how to install Jekyll on Windows using Vagrant.

1.  Install [Vagrant](http://www.vagrantup.com/downloads.html) 
2.  Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 
3.  Install the following Vagrant plugins:

    ``` bash
    vagrant plugin install vagrant-vbguest 
    ```

4.  Clone this git repo which contains the Vagrant configuration:

    ``` bash
    git clone https://github.com/greyfocus/vagrant-jekyll.git
    ```

5.  Launch Vagrant

	  ``` bash
    vagrant up
	  ``` 

## Workflow
 
1.  Use your favorite tools to edit the Jekyll files locally, in the Windows 
host
2.  Once complete, connect to Vagrant over ssh: 

    ``` bash
    vagrant ssh 
    ```
3.  Build the site (make sure you're in the shared folder, i.e. `/vagrant`) 

    ``` bash
    jekyll build 
    ```

You can also preview the site locally by running the builtin Jekyll server:

``` bash
jekyll serve --host 0.0.0.0
```

and open `http://127.0.0.1:4000/` in your favorite browser (on the Windows host)

Happy publishing!

