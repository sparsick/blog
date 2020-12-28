---
title: 'Generate P2 Repository From Maven Artifacts In 2017'
date: 2017-09-22
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
  - JVM
  - Maven Tycho
  - OSGi
tags:
  - eclipse rcp
  - equinox
  - maven repository
  - maven tycho
  - osgi
  - p2
  - p2-repository
# comment: false # Disable comment if false.
---

Some years ago, I wrote a [blog post](http://blog.sandra-parsick.de/2013/07/04/generate-p2-repository-from-maven-dependencies/) about how to generate a P2 repository based on Maven artifacts. That described approach is obsolete nowadays and I'd like to show a new approach that is based on the [p2-maven-plugin](https://github.com/reficio/p2-maven-plugin) that was created to solve exactly this problem.

P2-Maven-Plugin Integration in Maven Build Life Cycle
-----------------------------------------------------

First at all, we bind the p2-maven-plugin's goal `site` to the Maven's life cycle phase `package`. This goal is responsible for the generation of the P2 repository.

```xml
<plugin>
  <groupId>org.reficio</groupId>
  <artifactId>p2-maven-plugin</artifactId>
  <version>1.3.0</version>
  <executions>
    <execution>
      <id>default-cli</id>
      <phase>package</phase>
      <goals>
        <goal>site</goal>
      </goals>
      <!--... -->
    </execution>
  </executions>
</plugin>
```

Generating P2 Repository
------------------------

Now, we can define which Maven artifacts should be a part of the new P2 repository. It is irrelevant for the p2-maven-pluging if the defined artifacts have already a OSGi manifest or not. If no OSGi manifest exists, the plugin will generate one.

```xml
<execution>
<!-- ... -->
<configuration>
  <artifacts>
    <!-- specify your dependencies here -->
    <!-- groupId:artifactId:version -->
    <artifact>
      <id>com.google.guava:guava:jar:23.0</id>
      <!-- Artifact with existing OSGi-Manifest-->
    </artifact>
    <artifact>
      <id>commons-io:commons-io:1.3</id>
      <!-- Artifact without existing OSGi-Manifest-->
    </artifact>
  </artifacts>
</configuration>
</execution>
```

The artifacts are specified by the pattern `groupId:artifactId:version`. If you want to save some typing, use the `Buildr` tab on [MVN repository website](https://mvnrepository.com/) for copying the right dependency declaration format. ![MVN repository website](mvn.png?w=300) This sample configuration creates a P2 repository that look like the following one:

```text
target/repository
├── artifacts.jar
├── category.xml
├── content.jar
└── plugins
    ├── com.google.code.findbugs.jsr305_1.3.9.jar
    ├── com.google.errorprone.error_prone_annotations_2.0.18.jar
    ├── com.google.guava_23.0.0.jar
    ├── com.google.j2objc.annotations_1.1.0.jar
    ├── commons-io_1.3.0.jar
    └── org.codehaus.mojo.animal-sniffer-annotations_1.14.0.jar

1 directory, 9 files
```
 The default behavior of the plugin is, that all transitive dependencies of the defined artifact are also downloaded and packed into the P2 repository. If you don't want it, then you have to set the option `transitive` to `false` in the corresponded artifact declaration. If you need the sources (if they exist in the Maven repository) of the defined artifact in the P2 repository, then you have to set the option `source` to `true` in the corresponded artifact declaration.

```xml
<!-- ... -->
<artifact>
  <id>com.google.guava:guava:jar:23.0</id>
  <transitive>false</transitive>
  <source>true</source>
</artifact>
<!-- ... -->
```

Then the generated P2 repository looks like the following one:

```text
target/repository
├── artifacts.jar
├── category.xml
├── content.jar
└── plugins
    ├── com.google.guava.source_23.0.0.jar
    ├── com.google.guava_23.0.0.jar
    └── commons-io_1.3.0.jar

1 directory, 6 files
```

Generating P2 Repository With Grouped Artifacts
-----------------------------------------------

In some situations, you want to group artifacts in so-called _feature_. p2-maven-plugin provides an option that allows to group the Maven artifact directly into features. The definition of the artifacts is the same like above. The difference is that it has to be inside the corresponded feature. Then, the feature definition needs some meta data information like feature ID, feature version, description etc.

```xml
<!-- ...-->
<configuration>
  <featureDefinitions>
    <feature>
      <!-- Generate a feature including artifacts that are listed below inside the feature element-->
      <id>spring.feature</id>
      <version>4.3.11</version>
      <label>Spring Framework 4.3.11 Feature</label>
      <providerName>A provider</providerName>
      <description>${project.description}</description>
      <copyright>A copyright</copyright>
      <license>A licence</license>
      <artifacts>
        <artifact>
          <id>org.springframework:spring-core:jar:4.3.11.RELEASE</id>id>
        </artifact>
        <artifact>
          <id>org.springframework:spring-context:jar:4.3.11.RELEASE</id>id>
          <source>true</source>
        </artifact>
      </artifacts>
    </feature>
    <!--...-->
  </featureDefinitions>
  <!-- ... -->
<configuration>
```
Then the generated P2 repository looks like the following one:

```text
target/repository
├── artifacts.jar
├── category.xml
├── content.jar
├── features
│   └── spring.feature_4.3.11.jar
└── plugins
    ├── org.apache.commons.logging_1.2.0.jar
    ├── org.springframework.spring-aop.source_4.3.11.RELEASE.jar
    ├── org.springframework.spring-aop_4.3.11.RELEASE.jar
    ├── org.springframework.spring-beans.source_4.3.11.RELEASE.jar
    ├── org.springframework.spring-beans_4.3.11.RELEASE.jar
    ├── org.springframework.spring-context.source_4.3.11.RELEASE.jar
    ├── org.springframework.spring-context_4.3.11.RELEASE.jar
    ├── org.springframework.spring-core_4.3.11.RELEASE.jar
    ├── org.springframework.spring-expression.source_4.3.11.RELEASE.jar
    └── org.springframework.spring-expression_4.3.11.RELEASE.jar

2 directories, 14 files
```

Of course both options (generating p2 repository with feature and only with plugins) can be mixed. p2-maven-plugin provides more options like excluding specific transitive dependencies, referencing to other eclipse features and so on. For more information, please look at the p2-maven-plugin homepage. Now, we can generate P2 repositories from Maven artifacts. We lacks of how to deploy this P2 repository to a Repository manager like Artifactory or Sonatype Nexus. Both repository manager supports P2 repositories, Artifactory in the Professional variant (cost money) and Sonatype Nexus in OSS variant (free). For Nexus, it's important that you use the version 2.x. The newest version, 3.x, doesn't yet support P2 repositories.

Deploying P2 Repository to a Repository Manager
-----------------------------------------------

First at all, we want that our generated P2 repository is packed into a zip file. Therefore, we add the tycho-p2-repository-plugin to the Maven build life cycle:

```xml
<plugin>
  <groupId>org.eclipse.tycho</groupId>
  <artifactId>tycho-p2-repository-plugin</artifactId>
  <version>1.0.0</version>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>archive-repository</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

Then, we have to mark this zip file, so that Maven recognize that it has to deploy it during the deploy phase to a repository manager. For this, we add the build-helper-maven-plugin to the Maven build life cycle.

```xml
<!-- Attach zipped P2 repository to be installed and deployed in the Maven repository during the deploy phase. -->
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>build-helper-maven-plugin</artifactId>
  <version>3.0.0</version>
  <executions>
    <execution>
      <goals>
        <goal>attach-artifact</goal>
      </goals>
      <configuration>
        <artifacts>
          <artifact>
            <file>target/${project.artifactId}-${project.version}.zip</file>
            <type>zip</type>
          </artifact>
        </artifacts>
      </configuration>
    </execution>
  </executions>
</plugin>
```

Now, the generated P2 repository can be addressed by other projects. For more information about how to address the P2 repository, please have a look on the documentation of your repository manager. A whole pom.xml sample can be found on [Github](https://github.com/sparsick/generate-p2-repository-from-maven-artifacts).

Links
-----

* [Old blog post from 2013](http://blog.sandra-parsick.de/2013/07/04/generate-p2-repository-from-maven-dependencies/)
* [P2 Maven Plugin](https://github.com/reficio/p2-maven-plugin)
* Whole source code of the sample on [Github](https://github.com/sparsick/generate-p2-repository-from-maven-artifacts)
