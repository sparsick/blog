---
title: 'Set up a Proxy P2 Repository with Nexus OOS'
date: 2012-04-25
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
  - Nexus
tags:
  - how-to
  - nexus
  - p2-repository
  - proxy


# comment: false # Disable comment if false.
---


### Assumption

I assume [Nexus OOS in version 2.1.2](http://www.sonatype.org/nexus/go) is installed. You can find a good tutorial in Sonatype's Nexus book _Repository Management with Nexus_ ([Chapter 3: Installing and Running Nexus](http://www.sonatype.com/books/nexus-book/reference/install.html)).

### Preparation

For the set up of a proy P2 repository three Nexus plugins are needed:

*   Nexus Capability Plugin (It is contained  in the basic Nexus installation)
*   Nexus P2 Bridge Plugin 2.0.5 ([Download](https://repository.sonatype.org/index.html#nexus-search;quick~nexus-p2-bridge-plugin))
*   Nexus P2 Repository Plugin 2.2  ([Download](https://repository.sonatype.org/index.html#nexus-search;quick~nexus-p2-repository-plugin))

It is important that you download the artifacts ending with `-bundle.zip`. Unzip both plugins in the directory `$NEXUS_HOME/../sonatype-work/nexus/plugin-repository` of your Nexus instance. Restart your Nexus instance. Then follow the [instruction](http://www.sonatype.com/books/nexus-book/reference/_proxy_p2_repositories.html) for creating a proxy P2 repository in the Sonatype Nexus book.

### Troubleshooting

After I had created two proxy P2 Repositories, Nexus ran unstable. It restarted every night, automatically. A [post](http://maven.40175.n5.nabble.com/Failing-to-create-Proxy-Repository-for-P2-Update-Sites-td5537307.html "Nexus Mailing List Post") in Nexus Mailing List advised me to increasing the heap space to 1024MB for a stable run with proxies P2 Repository:

1.  Open the config file `$NEXUS_HOME/nexus/bin/jsw/conf/wrapper.conf`.
2.  Edit the property `wrapper.java.maxmemory`.

Increasing the heap space to `1024MB` solves my problem.

### Links

*   [Nexus OOS Download Site](http://www.sonatype.org/nexus/go "Nexus OOS Download Site")
*   [Sonatype's Nexus book _Repository Management with Nexus_](http://www.sonatype.com/books/nexus-book/reference/index.html "Nexus book")
*   [Nexus Mailing List](http://maven.40175.n5.nabble.com/Nexus-Maven-Repository-Manager-Users-List-f127899.html "Nexus Mailing List")
