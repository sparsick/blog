---
title: 'Read Classpath Resource from Jar Files with Spring'
date: 2012-02-29
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
  - Spring Framework
tags:
  - classpath
  - jar
  - resource
  - spring

# comment: false # Disable comment if false.
---

To read resources from classpath, you can use the Spring class [ClassPathResource](http://static.springsource.org/spring/docs/3.0.6.RELEASE/javadoc-api/org/springframework/core/io/ClassPathResource.html "JavaDoc Spring Framework ClassPathResource"). The constructor with one argument reads only resources from file system.  But sometimes you want to read a resources from a jar file in the classpath.  For this case, you must build an own [URLClassLoader](http://docs.oracle.com/javase/7/docs/api/java/net/URLClassLoader.html "JavaDoc Java 7 URLClassLoader") from the URL of the jar file and use the [ClassPathResource](http://static.springsource.org/spring/docs/3.0.6.RELEASE/javadoc-api/org/springframework/core/io/ClassPathResource.html "JavaDoc Spring Framework ClassPathResource") constructor with the arguments `path` and `classLoader`.  The `path` value is the path of the resource in the jar file.

### An Example

We have a jar file, named `example.jar`. The content looks as follows:

```shell
example.jar
  |_ META-INF
     |_ sample-resource.txt
```
We want to read the `sample-resource.txt`.  The source code for this example looks as follows:

```java
URL jarUrl = new File("path_to_jar_file").toURI().toURL();
URLClassLoader jarLoader = new URLClassLoader(new URL[]{jarUrl}, Thread.currentThread().getContextClassLoader());
ClassPathResource sampleResource = new ClassPathResource("META-INF/sample-resource.txt",jarLoader);
```
