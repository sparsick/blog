---
title: 'Groovy Script for Jenkins Script Console to Add or Replace Subversion Repository Browser'
date: 2013-10-15
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
  - subversion
  - sventon
# comment: false # Disable comment if false.
---

In my [last post](https://blog.sandra-parsick.de/2013/09/01/groovy-script-for-jenkins-script-console-to-rename-the-subversion-host-name/ "Groovy Script for Jenkins Script Console to Rename the Subversion Host Name") I showed how to rename the host name for Subversion in all jobs with one script. In this post I will show you how to set the repository browser in all jobs with one script. This script is based on the script from my [last post](https://blog.sandra-parsick.de/2013/09/01/groovy-script-for-jenkins-script-console-to-rename-the-subversion-host-name/ "Groovy Script for Jenkins Script Console to Rename the Subversion Host Name").

```groovy
def newRepositoryBrowserRootUrl = new URL("http://root.url.to.your.sventon.instance")
def newRepositoryInstance = "repository-instance-name"
def newRepositoryBrowser = new  hudson.scm.browsers.Sventon2(newRepositoryBrowserRootUrl, newRepositoryInstance)

Hudson.instance.items.each { item ->

    if(item.scm instanceof hudson.scm.SubversionSCM) {
        println("JOB: " + item.name)

        def newScm = new hudson.scm.SubversionSCM(Arrays.asList(item.scm.locations), item.scm.workspaceUpdater,
            newRepositoryBrowser, item.scm.excludedRegions, item.scm.excludedUsers, item.scm.excludedRevprop, item.scm.excludedCommitMessages,
            item.scm.includedRegions, item.scm.ignoreDirPropChanges, item.scm.filterChangelog)

        item.scm = newScm
        item.save()

        println("New Repository Browser: " +  item.scm.browser.class)
        println("\n=================\n")

    }
}
```

As aforementioned the above Groovy script uses Sventon 2.x as Subversion repository browser. However, Jenkins supports more Subversion repository browsers originally like

*   Assembla
*   CollabNetSVN
*   FishEyeSVN
*   SVNWeb
*   Sventon 1.x
*   ViewSVN
*   WebSVN

Jenkins supports other Subversion repository browsers by plugins like

*   Polarion WebClient  for Subversion
*   WebSVN 2.x

If you want to use an another Subversion repository browser, you have to change the first three lines:

```groovy
def newRepositoryBrowserRootUrl = new URL("http://root.url.to.your.sventon.instance")
def newRepositoryInstance = "repository-instance-name"
def newRepositoryBrowser = new  hudson.scm.browsers.Sventon2(newRepositoryBrowserRootUrl, newRepositoryInstance)
```

For example, if you want to use SVNWeb as Subversion repository browser, you have to add following lines instead of the aforementioned lines:

```groovy
def newRepositoryBrowserUrl = new URL("http://root.url.to.your.svn")
def newRepositoryBrowser = new hudson.scm.browsers.SVNWeb(newRepositoryBrowserUrl)
```

Links
-----

1.  [Blog Post - Groovy Script for Jenkins Script Console to Rename the Subversion Host Name](https://blog.sandra-parsick.de/2013/09/01/groovy-script-for-jenkins-script-console-to-rename-the-subversion-host-name/ "Groovy Script for Jenkins Script Console to Rename the Subversion Host Name")
2.  [Overview about supported Subversion repository browser by Jenkins](https://github.com/jenkinsci/subversion-plugin/tree/master/src/main/java/hudson/scm/browsers)
3.  [Polarion Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Polarion+Plugin)
4.  [WebSVN2 Plugin](https://wiki.jenkins-ci.org/display/JENKINS/WebSVN2+Plugin)
