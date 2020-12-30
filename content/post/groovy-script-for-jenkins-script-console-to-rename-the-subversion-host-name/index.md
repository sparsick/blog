---
title: 'Groovy Script for Jenkins Script Console to Rename the Subversion Host Name'
date: 2013-09-01
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
  - Jenkins
tags:
  - groovy
  - jenkins
  - subversion
# comment: false # Disable comment if false.
---
The host name of a Subversion instance is renamed and now many Jenkins jobs have to be adjusted. One possibility is to change every job by hand. But this approach is very time-consuming and error-prone. A better possibility is to have a script based approach, that rename the Subversion host name of all jobs in one go. Jenkins has a feature, that helps us thereby. Jenkins has a script console to run Groovy script on Jenkins server [1]. Below you can see a Groovy script  that renames the Subversion host name in all jobs.

```groovy
def oldHostName = "old.hostname.com"
def newHostName = "new.hostname.com"

Hudson.instance.items.each { item ->

  if(item.scm instanceof hudson.scm.SubversionSCM) {
    println("JOB : "+item.name)

    def newLocations = new ArrayList<hudson.scm.SubversionSCM.ModuleLocation>()

    item.scm.locations.each {location ->

      println("SCM Location Remote : " + location.remote)
      def newRemoteUrl = location.remote.replaceFirst(oldHostName, newHostName)

      newLocations.add(new hudson.scm.SubversionSCM.ModuleLocation(newRemoteUrl, location.local, location.depthOption,location.ignoreExternalsOption))
    }

    def newScm = new hudson.scm.SubversionSCM(newLocations, item.scm.workspaceUpdater,
    item.scm.browser, item.scm.excludedRegions, item.scm.excludedUsers, item.scm.excludedRevprop, item.scm.excludedCommitMessages,
    item.scm.includedRegions, item.scm.ignoreDirPropChanges, item.scm.filterChangelog)

    newScm.locations.each { location ->
      println("New SCM Location Remote : " + location.remote)
    }

    item.scm = newScm
    item.save()
    println("\n=======\n")
  }
}
```

This script is tested with Jenkins version 1.534 and Jenkins Subversion Plugin version 1.53.

### Links

[1] [Jenkins Script Console](https://wiki.jenkins-ci.org/display/JENKINS/Jenkins+Script+Console "Jenkins Script Console")
