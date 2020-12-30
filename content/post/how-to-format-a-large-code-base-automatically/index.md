---
title: 'How to Format a Large Code Base Automatically'
date: 2018-10-19
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
  - Git
  - Groovy
  - JVM
  - Quality Assurance
tags:
  - automation
  - code formatting
  - git
  - groovy
  - intellij
  - qa
# comment: false # Disable comment if false.
---
If you introduce code formatting rules retroactively, you have to solve the problem how to format existing code base according to the new formatting rules. You could checkout every code repository one by one in your IDE and click on `Autoformat the whole project.` But this is boring and waste of time. Fortunately, Intellij IDEA has a format CLI tool in its installation. You can locate it in the path `<your Intellij IDEA installation>/bin`. It's called `format.sh`. In the next section I'd like to show you how you can automate formatting big code base. First, I will show the preparation steps like exporting your code formatting rule setting from the IDE. Then, I will demonstrate how to use the CLI-Tool `format.sh.` At the end, I will show a small Groovy script that query all repositories (in this case they are Git repositories), formatting the code and push it back to the remote SCM.

Preparations
------------

First at all, we need the code formatting rule setting exported from Intellij IDEA. In your Intellij IDEA follow the next step

1.  Open `File -> Settings -Editor-> Code Style`
2.  Click on `Export...`
![](export-code-formatting-rules.png?w=300)
3.  Choose a name for the XML file (for example, `Default.xml`) and a location where this file should be saved (for example, `/home/foo` ).

Then, checkout or clone your SCM repository and remember the location where you checkout/clone it (for example, `/home/foo/myrepository`).

Format Code Base Via `format.sh`  CLI Tool
------------------------------------------

Three parameters are important for `format.sh:`

*   `-s :` Set a path to Intellij IDEA code style settings .xml file (in our example: `/home/foo/Default.xml`).
*   `-r :` Set that directories should be scanned recursively.
*   `path<n> :` Set a path to a file or a directory that should be formatted (in our example: `/home/foo/myrepository`).

```bash
> ./format.sh
IntelliJ IDEA 2018.2.4, build IC-182.4505.22 Formatter
Usage: format [-h] [-r|-R] [-s|-settings settingsPath] path1 path2...
-h|-help Show a help message and exit.
-s|-settings A path to Intellij IDEA code style settings .xml file.
-r|-R Scan directories recursively.
-m|-mask A comma-separated list of file masks.
path<n> A path to a file or a directory.

> /format.sh -r -s ~/Default.xml ~/myrepository
```

It's possible that the tool cancels scanning because of a `java.lang.OutOfMemoryError: Java heap space.` Then, you have to increase Java's maximum memory size `(-Xmx)` in `<your Intellij IDEA installation>/bin/idea64.vmoptions.`

```bash
> nano idea64.vmoptions
-Xms128m
-Xmx750m // <- here increase the maximum memory size
-XX:ReservedCodeCacheSize=240m
-XX:+UseConcMarkSweepGC
-XX:SoftRefLRUPolicyMSPerMB=50
-ea
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-Djdk.http.auth.tunneling.disabledSchemes=""
-XX:+HeapDumpOnOutOfMemoryError
-XX:-OmitStackTraceInFastThrow
-Dawt.useSystemAAFontSettings=lcd
-Dsun.java2d.renderer=sun.java2d.marlin.MarlinRenderingEngine
```

Groovy Script For Formatting Many Repository In a Row
-----------------------------------------------------

Now, we want to bring everything together. The script should do four things:

1.  Find all repository URLs whose code has to be formatted.
2.  Check out / Clone the repositories.
3.  Format the code in all branches of each repostory.
4.  Commit and push the change to the remote SCM.

I choose Git as SCM in my example. The finding of the repository URLs depends on the Git Management System (like BitBucket, Gitlab, SCM Manager etc.) that you use. But the approach is in all system the same:

1.  Call the RESTful API of your Git Management System.
2.  Parse the JSON object in the response after the URLs.

For example, in BitBucket it's like that:

```groovy
@Grab('org.codehaus.groovy.modules.http-builder:http-builder:0.7.1')
import groovyx.net.http.*

import static groovyx.net.http.ContentType.*
import static groovyx.net.http.Method.*

def cloneUrlsForProject() {

    def projectUrl = "https://scm/rest/api/1.0/projects/PROJECT_KEY/repos?limit=1000"
    def token = "BITBUCKET_TOKEN"

    def projects = []
    def cloneUrls = []

    def http = new HTTPBuilder(projectUrl)
    http.request(GET) {
        headers."Accept" = "application/json"
        headers."Authorization" = "Bearer ${token}"

        response.success = { resp -> projects = new JsonSlurper().parseText(resp.entity.content.text)}

        response.failure = { resp ->
            throw new RuntimeException("Error fetching clone urls for '${projectKey}': ${resp.statusLine}")
        }
    }

    projects.values.each { value ->
        def cloneLink = value.links.clone.find { it.name == "ssh" }
        cloneUrls.add(cloneLink.href)
    }

    return cloneUrls
}
```

Then, we have to clone the repositories and checkout each branch. In each branch, the `format.sh` has to be called. For the git operation, we use the jgit library and for the `format.sh` call we use a Groovy feature for process calling. In Groovy it's possible to define the command as a String and then to call the method `execute()` on this String like `"ls -l".execute()`. So the Groovy script for the last three tasks would be looked like that:

```groovy
#!/usr/bin/env groovy
@Grab('org.eclipse.jgit:org.eclipse.jgit:5.1.2.201810061102-r')
import jgit.*
import org.eclipse.jgit.api.CreateBranchCommand
import org.eclipse.jgit.api.Git
import org.eclipse.jgit.api.ListBranchCommand
import org.eclipse.jgit.transport.UsernamePasswordCredentialsProvider

import java.nio.file.Files


def intellijHome = 'path to your idea home folder'
def codeFormatterSetting = 'path to your exported code formatter setting file'
def allRepositoriesUrls = ["http://scm/repo1","http://scm/repo2"] // for simplifying

allRepositoriesUrls.each { repository ->
    def repositoryName = repository.split('/').flatten().findAll { it != null }.last()
    File localPath = Files.createTempDirectory("${repositoryName}-").toFile()
    println "Clone ${repository} to ${localPath}"
    Git.cloneRepository()
       .setURI(repository)
       .setDirectory(localPath)
       .setNoCheckout(true)
       .setCredentialsProvider(new UsernamePasswordCredentialsProvider("user", "password")) // only needed when clone url is https / http
       .call()
       .withCloseable { git ->
        def remoteBranches = git.branchList().setListMode(ListBranchCommand.ListMode.REMOTE).call()
        def remoteBranchNames = remoteBranches.collect { it.name.replace('refs/remotes/origin/', '') }

        println "Found the following branches: ${remoteBranchNames}"

        remoteBranchNames.each { remoteBranch ->
            println "Checkout branch $remoteBranch"
            git.checkout()
               .setName(remoteBranch)
               .setCreateBranch(true)
               .setUpstreamMode(CreateBranchCommand.SetupUpstreamMode.TRACK)
               .setStartPoint("origin/" + remoteBranch)
               .call()

            def formatCommand = "$intellijHome/bin/format.sh -r -s $codeFormatterSetting $localPath"

            println formatCommand.execute().text

            git.add()
               .addFilepattern('.')
               .call()
            git.commit()
               .setAuthor("Automator", "no-reply@yourcompany.com")
               .setMessage('Format code according to IntelliJ setting.')
               .call()

            println "Commit successful!"
        }

        git.push()
           .setCredentialsProvider(new UsernamePasswordCredentialsProvider("user", "password")) // only needed when clone url is https / http
           .setPushAll()
           .call()

        println "Push is done"

    }
}
```
Do you have another approach? Let me know and write a comment below.
