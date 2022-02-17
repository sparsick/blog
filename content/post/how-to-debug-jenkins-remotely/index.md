---
title: "How to Debug Jenkins remotely" # Title of the blog post.
date: 2022-02-17 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
#codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
#codeLineNumbers: false # Override global value for showing of line numbers within code block.
#figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Jenkins
  - Continuous Integration
tags:
  - jenkins
  - java
  - debugging
# comment: false # Disable comment if false.
---

Thanks Jenkins [Maven HPI Plugin](http://jenkinsci.github.io/maven-hpi-plugin/), you can do the most Jenkins plugin development stuff locally.
But in some cases, you have to debug a Jenkins instance remotely (In my case, the Jenkins had to run in Azure to reproduce a bug).

Jenkins is a Java application, so you have to add Java's JDWP agent for debugging as Java argument to Jenkins config

```text
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000 // for JDK 9 or later
```

The big question is, where is the configuration file on the machine to do it.

Answer: It depends on your operating system  🤓

Fortunately, Cloudbees has a [good overview](https://support.cloudbees.com/hc/en-us/articles/209715698-How-to-add-Java-arguments-to-Jenkins-) where to find the configuration files depend on the operating system .

In my case, I use Ubuntu as operating system. On Ubuntu, the configuration file is `/etc/default/jenkins`. I can extend the system environment variable `JAVA_ARGS`, there (see sample below).

```shell {hl_lines=[11]}
# defaults for Jenkins automation server

# pulled in from the init script; makes things easier.
NAME=jenkins

# arguments to pass to java

# Allow graphs etc. to work even when an X server is present
JAVA_ARGS="-Djava.awt.headless=true"

JAVA_ARGS="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000"

# more configuration stuff
```

Then restart your Jenkins instance (`sudo service jenkins restart`). Then you should see in Jenkins log (default on Ubuntu: `/var/log/jenkins/jenkins.log`) the following snippet:

```text
Listening for transport dt_socket at address: 8000
```

Now, you can connect your IDE via *Remote Debug* to your Jenkins instance.


### Links

* [Java's JDWP agent for debugging](https://docs.oracle.com/javase/8/docs/technotes/guides/jpda/conninv.html#Invocation)
* [Cloudbees' article about how to add Java arguments to Jenkins](https://support.cloudbees.com/hc/en-us/articles/209715698-How-to-add-Java-arguments-to-Jenkins-)
