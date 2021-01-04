---
title: 'How To Debug Groovy Script From Shell'
date: 2015-11-02
#description: "Article description." # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
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
  - Groovy
  - JVM
tags:
  - groovy
  - how-to
  - jvm
  - shell
# comment: false # Disable comment if false.
---

Groovy is a scripting language, so it is possible to run Groovy code without compiling to Java byte code. The necessary condition is that Groovy is installed on your machine. Then, running a Groovy script in a shell looks like the following line.

```bash
groovy TestScript.groovy
```

Now, something is wrong with the script, only on a special environment. So you want to debug your Groovy script from the shell. Fortunately, it works for Groovy just like for Java. You only have to export the Java options for debugging.

```bash
export JAVA_OPTS="-Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=4000,suspend=y"
```
Now, we can debug our script running from the shell with our favorite IDE.

```bash
groovy TestScript.groovy
Listening for transport dt_socket at address: 4000
```
