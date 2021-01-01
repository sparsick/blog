---
title: 'Release Preparing of a Maven Project'
date: 2011-12-08
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
  - Maven
tags:
  - maven
  - release

# comment: false # Disable comment if false.
---
This post describes how to prepare a release for Maven project with a set of Maven Plugins.

### Background

The release preparation phase is built of following common steps:

1.  Build the artifact to verify that the build is successful.
2.  Run the unit tests of the artifact.
3.  Update the version of the artifact to the release version.
4.  Commit the artifact to the SCM.
5.  Create a tag of the artifact.
6.  Update the version of the artifact to the next development version.
7.  Commit the artifact to the SCM.

These common steps must be extended when the artifact has snapshot dependencies. The snapshot dependencies  appears in following types:

*   The parent POM has a snapshot version.
*   The snapshot version of a group of dependencies is abstracted by a Maven propery.
*   The Maven project is a multi module project (special case of  "The parent pom has a snapshot version.").

Notice, that the above types can be appeared in combination.

### Precondition

A SCM command line tool must be installed.

### Approachs to the Release Preparation

#### The Maven project has no dependencies with snapshot version

This case relates to the above described common steps. Here the Maven Release Plugin helps:

1.  Check out the project from SCM.
2.  Call the goal `prepare` of the Maven Release Plugin:
```bash
mvn release:prepare -Dusername=scm.username -Dpassword=scm.password
```
3.  The Maven Release Plugin starts in an interactive mode with following questions:
    1.  Which release version should use.
    2.  Which name of the tag in the SCM.
    3.  Which development version should use after the tag.

#### Maven multi module project

The approach is similar to the case "The Maven project has no dependencies with snapshot version" only that the sub modules also must be released. In this case the Maven Release Plugin helps:

1.  Check out the project from SCM.
2.  Call the goal `prepare` of the Maven Release Plugin:
```bash
mvn release:prepare -Dusername=scm.username -Dpassword=scm.password -DautoVersionSubmodules=true
```
The option `autoVersionSubmodules=true` means that all sub modules should get the same relase version and then the development version as the root project.
3.  The Maven Release Plugin starts in an interactive mode with following questions:
    1.  Which release version should use.
    2.  Which name of the tag in the SCM.
    3.  Which development version should use after the tag.

#### The Maven project has a parent POM with a snapshot version

Notice, this case doesn't means the parent POM in sub modules of a multi module Maven project. The above described common step are extended by following steps:

1.  Check, whether the parent POM project must be released.
2.  Update the parent pom version to the current release version.
3.  Follow the common release preparation steps.
4.  After the tag, update the parent POM version to the new development version.

For these steps, the Maven Release Plugin, Versions Maven Plugin and the Maven SCM Plugin help:

1.  Check out the project from SCM.
2.  For the update the parent POM version, the goal `update-parent` of the Versions Maven Plugin helps:
```bash
mvn versions:update-parent -DgenerateBackupPoms=false
```
 The Versions Maven Plugin looks for the current release version of the parent POM in the repository and replaces the snapshot version by the found release version. The option `generatebackupPoms=false` means that the plugin should not generate backup for the old pom file.
3.  Commit this change to the SCM with the goal `checkin` of the Maven SCM Plugin:
```bash
mvn scm:checkin -Dusername=<scm.username> -Dpassword=<scm.password> -Dmessage="[maven-scm-plugin] checkin parent pom release version as a part of the release preparation" -Dincludes=pom.xml
```
 The commit is important because the Maven Release Plugin aborts the `prepare` goal when it finds diff between working copy and SCM.
4.  Then the above described common steps can be done with the well-known Maven Release Plugin:
```bash
mvn release:prepare -Dusername=<scm.username> -Dpassword=<scm.password>
```

5.  The last step is to udpate the parent pom to the current development version and to commit this change:
```bash
mvn versions:update-parent -DallowSnapshots=true mvn scm:checkin -Dusername=<scm.username> -Dpassword=<scm.password> -Dmessage="[maven-scm-plugin] checkin parent pom development version as a part of the release preparation" -Dincludes=pom.xml</pre>
```


#### The Maven project has dependencies with snapshot version grouped in a Maven property

In the this case the above described common steps are extended with following steps:

1.  Check, whether the dependencies must be released.
2.  Update the version defined in properties to the current release version.
3.  Execute the common release preparation steps.
4.  Update the version defined in properties to the current development version.

With our well-known Maven plugins it looks like this:
```bash
mvn versions:update-properties -DgenerateBackupPoms=false mvn scm:checkin -Dusername=<scm.username> -Dpassword=<scm.password> -Dmessage="\[maven-scm-plugin\] checkin DU dependencies release version as a part of the release preparation" -Dincludes=pom.xml mvn versions:update-properties -DallowSnapshots=true mvn scm:checkin -Dusername=<scm.username> -Dpassword=<scm.password> -Dmessage="\[maven-scm-plugin\] checkin DU dependencies development version as a part of the release preparation" -Dincludes=pom.xml
```


### Links

1.  [Maven Release Plugin Home Page](http://maven.apache.org/plugins/maven-release-plugin/ "Maven Release Plugin Home Page")
2.  [Versions Maven Plugin Home Page](http://mojo.codehaus.org/versions-maven-plugin/ "Versions Maven Plugin Home Page")
3.  [Maven SCM Plugin Home Page](http://maven.apache.org/scm/maven-scm-plugin/ "Maven SCM Plugin Home Page")
