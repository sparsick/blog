---
title: 'Automated Build of RCP Artifacts with Maven Tycho - A field report (Part 1)'
date: 2012-01-16
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
  - Maven Tycho
  - OSGi
tags:
  - build
  - eclipse rcp
  - maven tycho


# comment: false # Disable comment if false.
---
This post series should not be a tutorial for Maven Tycho. You can find many tutorials on the web (see for example [2] and [3]). This post series should be a field report about the problems during the introduction of Maven Tycho.

### Why Maven Tycho?

One requirement is that we can use Maven artifacts. So it seems obvious to search after a solution based on Maven. After a search on the web, two Maven plugins are noticed:

* PDE Maven Plugin ([7]): This plugin builds the RCP artifacts with the help of a generated Ant script by Eclipse. But this generated Ant script is error-prone.
* Maven Tycho Plugin ([1]): Very young plugin to build RCP artifacts (current version 0.13.0), but the community is very active. It tries to build RCP artifacts in the similar way than Eclipse.

### The Proceeding

During working the tutorials out the impression occurs that the main problem is to solve the dependency problem. So this is the first task for the introduction of Maven Tycho.

### Solution of the dependency problem

In Maven the dependencies are declared in the POM file. In Eclipse the dependencies are declared by a target definition file. The goal is to merge these both worlds. Maven Tycho enables Maven to build against a pre-defined target definition file.

The next question is where the dependencies come from. Maven's solution is the Maven Repository. Eclipse has its own solution, P2 repository. In Eclipse's target definition file you can specify a URL to a P2 repository and Eclipse searches after the needed dependencies, automatically.

A further requirement is that the Maven repository can still use. The reason is that our server artifacts are stored in a Maven repository and the communication between client and server is via Hessian, so some artifacts are needed by client and server. Furthermore, many third party libraries are only available in a Maven repository.

All examples in the tutorial work only with P2 repositories. But the introducing of a P2 repository is not an option, because then two repositories have to be administered.

#### First Approach - Define a Target Definition File with the Type 'Directory'

The idea of this approach is that you can define a folder with jars (that are the dependencies) in the target definition file. So it seems that the problem can be reduced to how I get the dependencies to this folder.

This problem can be solved by the Maven Dependency Plugin ([9]). It has the goal `copy-dependencies`. This goal copies all dependencies that are declared in the `<dependencies>`-element of the pom file to a declared output directory. The configuration of the dependency plugin could look like the following one:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-dependency-plugin</artifactId>
  <version>2.3</version>
  <executions>
    <execution>
      <id>copy-dependencies</id>
      <phase>process-resources</phase>
      <goals>
        <goal>copy-dependencies</goal>
      </goals>
      <configuration>
        <excludeTransitive>true</excludeTransitive>
        <outputDirectory>${project.build.directory}/plugins</outputDirectory>
      </configuration>
    </execution>
  </executions>
</plugin>
```

This solution works well in Eclipse, but not in with Maven's `mvn clean install`, because Maven Tycho doesn't support target type 'Directory', only the target type 'Software Site'. So this solution doesn't work for us.

#### Second Approach - A solution based on Maven's 'pom-first' approach

With Maven Tycho, you can define that the RCP artifacts should be built against the dependencies that are defined in the POM (old-known Maven way).

```
<?xml version="1.0" encoding="UTF-8"?>
<plugin>
  <groupId>org.eclipse.tycho</groupId>
  <artifactId>target-platform-configuration</artifactId>
  <version>0.13.0</version>
  <configuration>
    <pomDependencies>consider</pomDependencies>
  </configuration>
</plugin>

```

With Maven Tycho, the build works fine. But in Eclipse you cannot work in Eclipse-known way, because it has not the needed dependencies. I thought I could solve this problem with the Maven Eclipse Plugin and its goal `eclipse:eclipse`. This plugin generates all needed Eclipse files but the dependencies are generated in the `Java Build Path` named `Referenced Library`. But Eclipse needs the dependencies for the RCP plugin development in the `Java Build Path` named `Plug-in Dependencies`. Next try!

#### Third Approach - Nexus P2 Repository Plugin

A further research results that Nexus OSS should get a new plugin that can simulate the access to a Maven repository whether it would be a P2 repository (P2 proxy) (see [4]). This plugin is an experimental feature at the moment, so you can only use the SNAPSHOT version of this plugin. Unfortunately, the SNAPSHOT version (mid December 2011) that I used did not work with Nexus OSS version 1.9.2.2. But I think this would be a good idea for this problem.

#### **Fourth Approach - Generation of a Local P2 Repository**

A hint in the Tycho user mailing list (see [5]) helps me find this approach. It exits a Maven Tycho plugin (see [6]), that can generate a P2 repository from a directory with a bunch of bundles. In the first approach, we used the Maven Dependency Plugin to copy all needed dependencies in a directory. For this approach I combine these two plugins with two another plugins to build a local P2 repository.

By the Maven Dependency Plugin all dependencies are copied to a directory named `plugins`. The Tycho P2 Extra Plugin can generate from this directory a local P2 repository. Unfortunately, this plugin does not generate the needed `feature.jar`. But without this jar Eclipse does not recognize that we have here a P2 repository (an introduction to the P2 repository you can find in [10]). So we have to build this jar.

We create a new Maven project with the packaging option `jar`. In this project, there is only one file: `feature.xml`. In this xml file the name of the bundles of the `plugins` directory are stored. The next code snippet shows an example for this entry:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<plugin  id="org.eclipse.core.contenttype"  download-size="0"  install-size="0"  version="3.4.100.v20110423-0524"  unpack="false"/>
```

Then this new generated `feature.jar` are copied by the Maven Dependency Plugin to the directory `features`. Now with these two directories (`features` and `plugins`), the Tycho P2 Extra Plugin can generate a local P2 repository that are recognized by Eclipse.

The next problem is that the target definition file accepts only absolute path for the target type 'Software Site'. But we want a target definition file that is independent of the platform. So we save a target definition file with a token for the location of the local P2 repository in `src/main/resources`. During the Maven build this token are replaced by absolute platform-dependent path to the location of the local P2 repository. This replacement is done by the Maven Replacer Plugin ([11]). The next code snippet shows the whole plugin configuration for the above described solution:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-dependency-plugin</artifactId>
  <version>2.3</version>
  <executions>
    <execution>
      <id>copy-dependencies</id>
      <phase>process-resources</phase>
      <goals>
        <goal>copy-dependencies</goal>
      </goals>
      <configuration>
        <excludeTransitive>true</excludeTransitive>
        <outputDirectory>${project.build.directory}/source/plugins</outputDirectory>
      </configuration>
    </execution>
    <execution>
      <id>copy-feature</id>
      <phase>process-resources</phase>
      <goals>
        <goal>copy</goal>
      </goals>
      <configuration>
        <artifactItems>
          <artifactItem>
            <groupId>skm.p2</groupId>
            <artifactId>repository.feature</artifactId>
            <version>0.0.1-SNAPSHOT</version>
            <type>jar</type>
            <overWrite>true</overWrite>
            <outputDirectory>${project.build.directory}/source/features</outputDirectory>
          </artifactItem>
        </artifactItems>
      </configuration>
    </execution>
  </executions>
</plugin>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-resources-plugin</artifactId>
  <version>2.5</version>
  <executions>
    <execution>
      <id>copy-target-definition</id>
      <phase>process-resources</phase>
      <goals>
        <goal>copy-resources</goal>
      </goals>
      <configuration>
        <resources>
          <resource>
            <directory>src/main/resources</directory>
            <includes>
              <include>${target.definition}</include>
            </includes>
          </resource>
        </resources>
        <outputDirectory>${project.build.directory}</outputDirectory>
      </configuration>
    </execution>
  </executions>
</plugin>
<plugin>
  <groupId>com.google.code.maven-replacer-plugin</groupId>
  <artifactId>maven-replacer-plugin</artifactId>
  <version>1.4.0</version>
  <executions>
    <execution>
      <phase>prepare-package</phase>
      <goals>
        <goal>replace</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <ignoreMissingFile>false</ignoreMissingFile>
    <file>target/${target.definition}</file>
    <regex>false</regex>
    <replacements>
      <replacement>
        <token>$repository.location$</token>
        <value>${project.build.directory}/repository/</value>
      </replacement>
      <replacement>
        <token>
        </token>
        <value>/</value>
      </replacement>
    </replacements>
  </configuration>
</plugin>
<plugin>
  <groupId>org.eclipse.tycho.extras</groupId>
  <artifactId>tycho-p2-extras-plugin</artifactId>
  <version>0.13.0</version>
  <executions>
    <execution>
      <phase>prepare-package</phase>
      <goals>
        <goal>publish-features-and-bundles</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <compress>false</compress>
  </configuration>
</plugin>
<plugin>
  <groupId>org.eclipse.tycho</groupId>
  <artifactId>tycho-p2-repository-plugin</artifactId>
  <version>0.13.0</version>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>archive-repository</goal>
      </goals>
    </execution>
  </executions>
</plugin>
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>build-helper-maven-plugin</artifactId>
  <version>1.7</version>
  <executions>
    <execution>
      <id>attach-artifacts</id>
      <phase>package</phase>
      <goals>
        <goal>attach-artifact</goal>
      </goals>
      <configuration>
        <artifacts>
          <artifact>
            <file>target/${target.definition}</file>
            <type>target</type>
            <classifier>indigo</classifier>
          </artifact>
        </artifacts>
      </configuration>
    </execution>
  </executions>
</plugin>
```

In Eclipse, this generated target definition file works fine, but in Maven Tycho it does not work: Before Maven Tycho builds the local P2 repository and the target definition file, it tries to build the RCP plugins against the not yet existing target definition file and the build failed.

#### **Fifth Approach and Final Solution - Combination of Dependency in the POM and Target Definition**

We know Maven works with in the POM defined dependencies, perfectly and Eclipse works with the target platform, perfectly. So the idea is to combine this two approaches, so that every tool can work in its perfect way. So I combine the first both approaches that are described in the above chapters.

All dependencies that are used by all RCP plugins are defined in the root project POM, so that every sub module that defines one RCP plugin is built against these dependencies and one sub module is for the building of the target definition file (this build is described in the first approach). Maven Tycho builds the RCP artifacts in the known Maven way and in Eclipse the RCP artifacts are developed with the generated target definition file.

Links
-----

[1] [Maven Tycho Home Page](http://www.eclipse.org/tycho/ "Maven Tycho Home Page")  
[2] [Tutorial by Matthias Holmqvist - Good for the first introduction](http://mattiasholmqvist.se/2010/02/building-with-tycho-part-1-osgi-bundles/)  
[3] [Tutorial with example (in German)](http://it-republik.de/jaxenter/artikel/Eclipse-RCP-Anwendungen-mit-Maven-und-Tycho-bauen-4156.html)  
[4] [Nexus P2 Repository Plugin (Experimental Feature)](https://docs.sonatype.org/display/Nexus/Nexus+OSGi+Experimental+Features+-+P2+Repository+Plugin)  
[5] [Post on Tycho User Mailing List - How to build local P2 Repository](http://dev.eclipse.org/mhonarc/lists/tycho-user/msg01422.html "Post Tycho Mailing List")  
[6] [Introduction to the Tycho-P2-Extra-Plugin](https://docs.sonatype.org/display/TYCHO/Tycho-extras+-+FeaturesAndBundlesPublisher "Tycho P2 Extra Plugin")  
[7] [PDE Maven Plugin](http://mojo.codehaus.org/pde-maven-plugin/ "PDE Maven Plugin")  
[8] [Hessian Protocol](http://en.wikipedia.org/wiki/Hessian_%28web_service_protocol%29 "Hessian Protocol")  
[9] [Maven Dependency Plugin](http://maven.apache.org/plugins/maven-dependency-plugin/ "Maven Dependency Plugin")  
[10] [Introduction to the format of P2 repositoy](http://eclipse.org/equinox/p2/repository_packaging.html#Repository "Format p2 repository")  
[11] [Maven Replacer Plugin](http://code.google.com/p/maven-replacer-plugin/ "Maven Replacer Plugin")
