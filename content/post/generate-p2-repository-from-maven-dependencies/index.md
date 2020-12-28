---
title: 'Generate P2 Repository From Maven Artifacts In 2017'
date: 2013-07-04
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
  - Continuous Integration
  - JVM
  - Maven
  - Maven Tycho
  - Nexus
  - OSGi
tags:
  - eclipse rcp
  - maven
  - maven tycho
  - nexus
  - p2
  - p2-repository
# comment: false # Disable comment if false.
---

_A new version of this blog post is published under the title "[Generate P2 Repository From Maven Artifacts in 2017](http://blog.sandra-parsick.de/2017/09/22/generate-p2-repository-from-maven-artifacts-in-2017/)"_

### Motivation

If you use Maven Tycho to build your Eclipse RCP application, you can use Maven dependencies as you know it from Maven (for more information see [1]). I notice that Tycho takes many times for computing the target platform from the Maven dependencies during the build. I expect a speed up of build time if I use a P2 repository instead of Maven dependencies directly. Therefore, the task is how do I get the Maven dependencies to a P2 repository.

### Procedure for generating a P2 repository from Maven dependencies

We generate the P2 repository with the help of some Maven's plugin and some Maven Tycho's plugin:

*   Builder Helper Maven Plugin
*   Maven Dependency Plugin
*   Maven Tycho's P2 Extra Plugin
*   Maven Tycho's P2 Plugin
*   Maven Tycho's P2 Repository Plugin

Maven Tycho's Plugins use Eclipse standard tools internally. How the Eclipse standard tools work are described well on this blog post [2]. Further useful information about Tycho's plugins can be found in [3] and [4]. We create a POM project and configure plugins mentioned above, so that the following procedure can work:

1.  Define the Maven dependencies, that should be add to the P2 repository, in the _<dependencies>_ section.
2.  Copy these defined dependencies to the source location of the Feature and Bundle Publisher with _Maven Dependency Plugin_.
3.  Generate P2 repository with _P2 Extra Plugin_.
4.  Add categories to the P2 metadata with _P2 Plugin_, so that you can see your P2 repository in Eclipse Target Platform Wizard.
5.  Zip P2 repository with _P2 Repository Plugin_.
6.  Attach zipped P2 repository to be installed and deployed in the Maven repository during the deploy phase with _Builder Helper Plugin_.

You can find the whole POM configuration in [5].

### How to use zipped P2 Repository from Maven Repository

When you use Nexus as Maven Repository, you can use the Nexus Unzip Plugin ([6]).

### Links

- [1] [Maven Tycho How to - Dependency on pom-first artifacts](http://wiki.eclipse.org/Tycho/How_Tos/Dependency_on_pom-first_artifacts)
- [2] [Blog Post about Generation P2 Repository with Eclipse Standard Tool](http://maksim.sorokin.dk/it/2010/11/26/creating-a-p2-repository-from-features-and-plugins/)
- [3] [P2 Extra Plugin Project Site](http://www.eclipse.org/tycho/sitedocs-extras/tycho-p2-extras-plugin/plugin-info.html)
- [4] [Sonatype Wiki Page about P2 Extra Plugin](https://docs.sonatype.org/display/TYCHO/Tycho-extras+-+FeaturesAndBundlesPublisher)
- [5] [POM Project on GitHub](https://github.com/sparsick/generate-p2-repository-from-maven-artifacts/tree/pre-p2-maven-plugin-solution)-
- [6] [Nexus Unzip Plugin Documentation](http://wiki.eclipse.org/Tycho/Nexus_Unzip_Plugin)
