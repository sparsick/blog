---
title: 'Groovy Script for Jenkins Script Console to Add or Modify Discard Old Builds Setting'
date: 2014-02-16
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
  - Continuous Integration
  - Jenkins
tags:
  - groovy
  - jenkins
# comment: false # Disable comment if false.
---

When your Jenkins server runs on for a while, your hard disk usage grows up because the default setting of each job is that every job build and build artifact obtained from this are stored in a build history.  Limiting this build history can save hard disk usage. Each job has a setting option called `Discard Old Builds`. In this setting you can set how long (in days) old builds keep in the build history (criteria 1) or how many old builds keep at most in the build history (criteria 2). Jenkins matches first criteria 1 and then criteria 2. Beyond that there exists advanced setting options. This advanced setting options can configured how long and how many builds with build artifacts have to keep in the build history. After that Jenkins deletes the build artifact, but the logs, history, reports, etc for the build will be kept. These will be kept so as long as they match the criteria in the normal setting options. When you have many jobs, you don't want to configure every job manually. For that you can modify the `Discard Old Builds` setting in each job over a Groovy script at one go. Jenkins has a so-called _Script Console_ [1].  In this console you have to put the following Groovy script and every job is modified to discard its old builds.

```groovy
def daysToKeep = 28
def numToKeep = 10
def artifactDaysToKeep = -1
def artifactNumToKeep = -1

Jenkins.instance.items.each { item ->
  println("=====================")
  println("JOB: " + item.name)
  println("Job type: " + item.getClass())

  if(item.buildDiscarder == null) {
    println("No BuildDiscarder")
    println("Set BuildDiscarder to LogRotator")
  } else {
    println("BuildDiscarder: " + item.buildDiscarder.getClass())
    println("Found setting: " + "days to keep=" + item.buildDiscarder.daysToKeepStr + "; num to keep=" + item.buildDiscarder.numToKeepStr + "; artifact day to keep=" + item.buildDiscarder.artifactDaysToKeepStr + "; artifact num to keep=" + item.buildDiscarder.artifactNumToKeepStr)
    println("Set new setting")
  }

  item.buildDiscarder = new hudson.tasks.LogRotator(daysToKeep,numToKeep, artifactDaysToKeep, artifactNumToKeep)
  item.save()
  println("")

}
```
This script is tested with Jenkins version 1.534 and Jenkins Subversion Plugin version 1.53. In my last posts ([2], [3]) I showed two other use cases for the _Jenkins Script Console._ All Groovy scripts can be found in GitHub [4]

### Links

- [1] [Jenkins Script Console](https://wiki.jenkins-ci.org/display/JENKINS/Jenkins+Script+Console "Jenkins Script Console")
- [2] [Post about how to rename Subversion host name in every job](https://blog.sandra-parsick.de/2013/09/01/groovy-script-for-jenkins-script-console-to-rename-the-subversion-host-name/ "Groovy Script for Jenkins Script Console to Rename the Subversion Host Name")
- [3] [Post about how to add or modify Subversion repository browser in every job](http://blog.sandra-parsick.de/2013/10/15/groovy-script-for-jenkins-script-console-to-add-or-replace-subversion-repository-browser/ "Groovy Script for Jenkins Script Console to Add or Replace Subversion Repository Browser")
- [4] [GitHub repository with several Groovy scripts for Jenkins script console](https://github.com/sparsick/scripts-jenkins-console/tree/1.x-branch)
