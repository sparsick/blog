---
title: 'Migration Path for Jenkins Subversion Plugin from Version 1.53 to 2.2'
date: 2014-03-21
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

Introduction
------------

Jenkins Subversion Plugin changes its credential management. Before version 2.2 you had to set your Subversion credential globally depend on domain name. In version 2.2 it's changed. The credential management is still central but now you have to configure in every job which credential the job has to use for Subversion authentication . If you have many jobs, you don't want to touch every job to change the Subversion configuration. A Groovy script for Jenkins Script Console [1] can help. In the next section I will describe how the migration path looks like.

Migration Path
--------------

1.  Update the Subversion Plugin in your Jenkins instance to version 2.2.
2.  Add your Subversion credential to the global credential store:
    1.  Go to `Jenkins -> Credential -> Global credentials -> Add credential`
    2.  Add your credential. The scope has to be `global.`
3.  Go to the installation path of your Jenkins instance and open the file `credential.xml.`
4.  Search in `credential.xml` for the element `<com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl>` that contains your above set credential and copy the value of the element `<id>` . Example:
```xml
<com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl>
  <scope>GLOBAL</scope>
  <id>63c3793c-e3fb-49eb-b45b-f6f8e7364876</id>
  <description></description>
  <username>jenkinssvnuser</username>
  HyDUenzpyDkbL9xMoQ0pxdK10l20VkXKEiy4+ZnjL9c=
</com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl>
```
5.  Go to the Jenkins Script Console and run following Groovy script. Assign your credential id to the variable `credentialId` before running the script.
```groovy
def credentialId = '63c3793c-e3fb-49eb-b45b-f6f8e7364876'

Hudson.instance.items.each { item ->

if(item.scm instanceof hudson.scm.SubversionSCM) {
  println("JOB : "+item.name)

  def newLocations = new ArrayList()

  item.scm.locations.each {location ->
  newLocations.add(new hudson.scm.SubversionSCM.ModuleLocation(location.remote, credentialId, location.local, location.depthOption,location.ignoreExternalsOption))
}

def newScm = new hudson.scm.SubversionSCM(newLocations, item.scm.workspaceUpdater,
item.scm.browser, item.scm.excludedRegions, item.scm.excludedUsers, item.scm.excludedRevprop, item.scm.excludedCommitMessages,
item.scm.includedRegions, item.scm.ignoreDirPropChanges, item.scm.filterChangelog, item.scm.additionalCredentials)

item.scm = newScm
item.save()
println("\n=======\n")
}
```
6.  Finish.

This script is tested with Jenkins version 1.555 and Jenkins Subversion Plugin version 2.2. On Github [2] you can find more Groovy script for Jenkins Script Console.

Links
-----

- [1] [Jenkins Script Console](https://wiki.jenkins-ci.org/display/JENKINS/Jenkins+Script+Console)
- [2] [More Groovy Script for Jenkins on Github](https://github.com/skosmalla/scripts-jenkins-console)
