---
title: 'Migration Sonatype Nexus 2 to Nexus 3'
date: 2016-12-30
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
  - Nexus
tags:
  - bower repository
  - maven repository
  - migration
  - nexus
  - repository-manager
# comment: false # Disable comment if false.
---

I'd like to share my experience with migration Sonatype Nexus 2 to Nexus 3.

Starting Point
--------------

I used two Nexus instances:

*   A Nexus 2 Instance for Maven Repositories (2.13)
*   A Nexus 3 Instance for Bower Repositories (3.0.1)

Both instances had several types of repositories (host, proxy, group). The reason for this set up was that Sonatype recommended not to use Nexus 3 (pre 3.1) for Maven repositories and Nexus 2 doesn't support Bower repositories.

Migration Path
--------------

1.  Update Nexus 2 instance to version 2.14.1 ([Update Guide](https://support.sonatype.com/hc/en-us/articles/213464198-How-do-I-upgrade-Nexus-))
2.  Update Nexus 3 Instance to version 3.1 ([Update Guide](https://support.sonatype.com/hc/en-us/articles/231723267) ). It's important that you migrate to the new working directory layout.
3.  Follow the migration step for upgrading from version 2 to version 3 ([Update Guide](https://books.sonatype.com/nexus-book/reference3/upgrading.html#upgrade-version-two-three)).
    *   My scenario was Nexus 2 and Nexus 3 running on the same system.
    *   I selected as `Upgrade Method` "File system copy"
    *   I chose that only "repository configuration content" has to migrate.
4.  After a successful migration I had to adjust some configuration in Nexus and in the system that use Nexus:
    *   Setup in Nexus 3 a new user for deploying artifacts.
    *   Adjust URL to the repositories in Jenkins, Maven settings and deployment scripts.
