---
title: 'Reading tips for OSGi Beginner'
date: 2011-09-25
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
  - Java
  - OSGi
tags:
  - ops4j
  - osgi
  - pax


# comment: false # Disable comment if false.
---

In the last weeks I familiarised myself with OSGi. Two books were very helpful:

* **Modular Java: Creating Flexible Applications with OSGi and Spring** by Craig Walls
* **OSGi für Praktiker - Prinzipien, Werkzeuge und praktische Anleitungen auf dem Weg zur kleinen SOA** by Bernd Weber,  Patrick Baumgartner, Oliver Braun (in German)

Both books are similar organised.  They introduce OSGi by an example project form the first source code till the deployment.  These projects are not 'Hello World' projects, so you can adopt the way of working with OSGi for your real OSGi projects.

After the books show the classic way to coding with OSGi, they show how you can use dependency injection in OSGi applications with Spring DM.

Both books work with the [PAX Tooling of OPS4J](http://team.ops4j.org/wiki/ "OPS4J Team"). They are Maven Plugins and track the 'pom-first' way to build OSGi applications.  These books have a good introduction to the PAX tooling, too. It is very helpful, when you want to use a continuous integration environment for your development (Of course, you want :) ).

The book 'Modular Java' has one minor flaw. It was written in 2009, so it use elder version of the tools.
