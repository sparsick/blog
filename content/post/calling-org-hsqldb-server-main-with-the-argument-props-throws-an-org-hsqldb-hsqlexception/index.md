---
title: 'Calling org.hsqldb.Server.main with the argument  "--props" throws an org.hsqldb.HsqlException'
date: 2011-10-18
#description: "Article description." # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Java
  - JVM
tags:
  - hsqldb

# comment: false # Disable comment if false.
---

Whensoever, you call, for example,  `java -cp ../lib/hsqldb.jar org.hsqldb.Server --props ../bin/config/server.properties` following exception is thrown:
```shell
[Server@e0e1c6]: [Thread[main,5,main]]: checkRunning(false) entered
[Server@e0e1c6]: [Thread[main,5,main]]: checkRunning(false) exited
[Server@e0e1c6]: [Thread[main,5,main]]: Failed to set properties
org.hsqldb.HsqlException: no valid database paths: unsupported property: server.props
at org.hsqldb.error.Error.error(Unknown Source)
at org.hsqldb.error.Error.error(Unknown Source)
at org.hsqldb.server.Server.setProperties(Unknown Source)
at org.hsqldb.server.Server.main(Unknown Source)
```

Debugging results in following: The main method finds the `server.properties` file and it can read the properties in this file, too.  But then the main method merges the properties from the file with the properties from the arguments (local variable named `argProps`).

```java
String propsPath = argProps.getProperty(ServerProperties.sc_key_props);
String propsExtension = "";

if (propsPath == null) {
  propsPath      = "server";
  propsExtension = ".properties";
}

propsPath = FileUtil.getFileUtil().canonicalOrAbsolutePath(propsPath);

ServerProperties fileProps = ServerConfiguration.getPropertiesFromFile(ServerConstants.SC_PROTOCOL_HSQL, propsPath, propsExtension);
ServerProperties props = fileProps == null ? new ServerProperties(ServerConstants.SC_PROTOCOL_HSQL) : fileProps;

props.addProperties(argProps);

```

But the `argProps` has still the property named `system.props=../bin/config/server.properties` and this property is for the server an invalid property, so in the method `Server.setProperties(HsqlProperties)` the calling of `HsqlProperties.validate()` is thrown the `org.hsqldb.HsqlException`.

## Solution

I replace the above if-clause by the following one:

```java
if (propsPath == null) {
  propsPath      = "server";
  propsExtension = ".properties";
} else {
  argProps.removeProperty(ServerProperties.sc_key_props);
}
```

That means, if a `propsPath` is found, the `props` property is in the `argProps`. So it can be removed. After this change the server starts as described as in the documentation. I found this problem in the current version 2.2.5, so I sent a patch to HSQLDB.

## Links

1. ~~[Patch Tracker Id of HSQLDB for this problem](http://sourceforge.net/tracker/?func=detail&aid=3425397&group_id=23316&atid=378133)~~ The issue was moved to the bug tracker. Here the new [link](http://sourceforge.net/tracker/?func=detail&aid=3425397&group_id=23316&atid=378131).

## Update
The bug will fix in the next release.
