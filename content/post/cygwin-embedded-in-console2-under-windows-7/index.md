---
title: 'Cygwin Embedded In Console2 Under Windows 7'
date: 2012-11-11
#description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Linux
tags:
  - bash
  - cmd
  - console2
  - cygwin
  - linux
  - Linux
  - shell
  - Windows

# comment: false # Disable comment if false.
---

I like to use [Cygwin](http://www.cygwin.com/) for having a bash shell on a windows machine. In combination with [Console2](http://sourceforge.net/projects/console/) you have a powerful command-line tool for windows. I'd like to describe how to install Cygwin and Console2 under windows 7 (I think this instruction works on Windows XP, too) and how to embed Cygwin in Console2. I used for Cygwin version 1.7.17-1 32bit and for Console2 version 2.00b148-Beta_32bit.

### Cygwin Installation

1.  Download [setup.exe.](http://cygwin.com/setup.exe)
2.  Double click on setup.exe starts the installation.
3.  Follow the set up instruction. As installation location I use `c:\cygwin.`
4.  Go to `c:\cygwin` and run `Cygwin.bat` for setting up Cygwin user home etc.

### Console2 Installation

1.  Download [zip file.](http://sourceforge.net/projects/console/files/console-devel/2.00/Console-2.00b148-Beta_32bit.zip/download)
2.  Unzip it into installation location.
3.  Start Console2 with a double click on `Console2.exe.`

### How to embed Cygwin in Console2

1.  Open Console2.
2.  Go to `Edit -> Settings.`
3.  Go to  `Tab`
4.  In field `title` insert for example `bash`. It is the name of the first tab in Console2.
5.  In field `Shell` you have to insert `C:\cygwin\bin\bash.exe --login -i` (Don't forget `c:\cygwin` is your installation folder).
6.  In field `Startup dir` you can insert `C:\cygwin\home\user.name.` This folder is used at every Console2 start.

### Links

1.  [Cygwin Homepage](http://www.cygwin.com/)
2.  [Console2 Homepage](http://sourceforge.net/projects/console/)
3.  [Good short feature overview of Cygwin on the Wikipedia page](http://en.wikipedia.org/wiki/Cygwin)
