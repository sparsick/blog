---
title: 'Distinctive Feature of Running Nexus on Linux'
date: 2011-09-29
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
  - Continuous Integration
  - Nexus
tags:
  - ibm
  - jvm
  - linux
  - nexus

# comment: false # Disable comment if false.
---

I installed Nexus on Linux according to the step-by-step instruction of Sonatype. Everything looked fine during the installation. To find out, whether the installation was successful, I tried to go to the welcome page of my nexus installation. I got an error message. I looked in the log file of Nexus and found following exception:

```text
com.google.inject.internal.ComputationException: com.google.inject.internal.ComputationException: java.lang.TypeNotPresentException: Type org.codehaus.enunciate.contract.jaxrs.ResourceMethodSignature not present
 ....

ERROR [er_start_runner] - /nexus - unavailable java.lang.IllegalStateException: The PlexusServerServlet couldn't lookup the target component (role='org.restlet.Application', hint='nexus')
```
With Google's help I found a fixed bug in the Nexus bug tracking system, that describes my problem. The reason for my problem was: I had on my Linux machine an IBM JVM in an elder version. With this version Nexus doesn't run because of a bug in the IBM JVM.  Two solutions and one workaround exist for this problem:

* _Workaround:_ Copying enunciate-core-*.jar to /runtime/apps/nexus/lib.
* _First solution:_ Update the version of IBM JVM to 5.0.0 SR12 / 6.0.0 SR8 FP1 or later
* _Second solution:_ Use the JVM of Oracle.

I decided to change the JVM to an Oracle JVM and after that, Nexus ran without errors.

## Links

1.  [Installation Guide for Nexus by Sonatype](http://www.sonatype.com/books/nexus-book/reference/install.html "Installation Guide for Nexus by Sonatype")
2.  [Bug reported to Nexus ](https://issues.sonatype.org/browse/NEXUS-3509 "bug nexus")
3.  [Bug reported to IBM](http://www-01.ibm.com/support/docview.wss?uid=swg1IZ76352 "bug ibm")
