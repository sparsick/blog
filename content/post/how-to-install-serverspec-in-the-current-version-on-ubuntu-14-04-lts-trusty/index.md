---
title: 'How to Install Serverspec in the Current Version on Ubuntu 14.04 LTS (Trusty)'
date: 2016-03-16
#description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
#codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Configuration Management
  - Linux
  - Testing
tags:
  - how-to
  - linux
  - ruby
  - serverspec
  - ubuntu
# comment: false # Disable comment if false.
---

If you google "serverspec install ubuntu", you find the information that a package called `ruby-serverspec` in the standard package repository can be used to install Serverspec on an Ubuntu 14.04 LTS based system. Unfortunately, this package installs an outdated version of Serverspec. The next point is that if you try to install the newest version of Serverspec with gem (that's the way that it is described on the Serverspec homepage), you will get the following error message:

```bash
~> sudo gem install serverspec
ERROR:  Error installing serverspec:
net-ssh requires Ruby version >= 2.0.
```

The problem is, when you install Ruby with `sudo apt-get install ruby`, the package manager installs Ruby in the version 1.9.1 . Therefore, the next sections explain how to install Ruby and Serverspec in the newest version on an Ubuntu 14.04 LTS based system. Let's start with Ruby that is required for Serverspec.

Ruby Installation
=================

The cloud hosting service Brightbox provides Ruby package repositories for several Ubuntu versions and several Ruby version. I chose the repository for Ruby 2.3 packages, so the installation steps are:

```bash
~> sudo apt-get install software-properties-common
~> sudo apt-add-repository ppa:brightbox/ruby-ng
~> sudo apt-get update
~> sudo apt-get install ruby2.3
~> ruby --version
ruby 2.3.0p0 (2015-12-25 revision 53290) [x86_64-linux-gnu]
```

Serverspec Installation
=======================

Now, we can install Serverspec like it is explained on the Serverspec homepage. In my case, I had to install `rake` separately.

```bash
~> sudo gem install serverspec rake
```

Links
=====

1.  [Serverspec Homepage](http://serverspec.org/)
2.  Brightbox Ruby package repositories for Ubuntu [documentation](https://www.brightbox.com/docs/ruby/ubuntu/)
