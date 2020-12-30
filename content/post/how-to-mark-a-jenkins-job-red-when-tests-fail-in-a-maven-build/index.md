---
title: 'How to Mark a Jenkins Job Red When Tests Fail In A Maven Build'
date: 2017-10-14
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
  - Maven
tags:
  - jenkins
  - maven
  - test-report
  - unit-testing
# comment: false # Disable comment if false.
---
The default setting in Jenkins is to mark a job yellow, when a Maven build fails because of failing tests. If you don't want to have three status of your jobs, you can configure Jenkins so, that the jobs also mark red independent why a Maven build fails. For this you will need administration rights on your Jenkins instance. Following steps have to be done:

1.  Go to `Manage Jenkins -> Manage system.`
2.  Add `-Dmaven.test.failure.ignore=false` to `Maven Project Configuration -> Global Maven_OPTS.`
3.  Save this change and that's it.

Your next job run will consider this configuration. Unfortunately, this configuration has only effects for Maven jobs. Freestyle jobs ignore this configuration (see also [this bug](https://issues.jenkins-ci.org/browse/JENKINS-24655)). But a workaround exists:

1.  Install the [TextFinder plugin](https://wiki.jenkins.io/display/JENKINS/Text-finder+Plugin) via `Manage Jenkins -> Manage Plugin.`
2.  Open the Freestyle job's configuration that should be marked red, when Maven tests fail.
3.  Click on `Add a post-build action `(in section `Post-build Action`) and select `Jenkins Text Finder.`
4.  Activate the check box `Also search the console output.`
5.  Add the value `There are test failures` to `Regular expression.`
6.  Save this change.
