---
title: 'How to Measure Test Coverage in Invoker Tests with JaCoCo'
date: 2021-05-31
description: "This article describes how to configure JaCoCo, so that the test coverage can be measured in invoker tests." # Description used for search engine.
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
  - Maven
  - Testing
  - Quality Assurance
tags:
  - maven
  - test-report
  - integration-testing
  - maven-plugin
  - maven-plugin-testing
  - maven-invoker-plugin
  - jacoco
  - test-coverage
# comment: false # Disable comment if false.
---

During the development of the [P2 Maven Plugin](https://github.com/reficio/p2-maven-plugin), I wanted to get an overview about the test coverage of the code.
The tests of this plugin can be divided in two groups:
- *Unit tests*, that is written with Java, Groovy and JUnit;
- *integration tests*, that is written with [Maven Invoker Plugin](https://maven.apache.org/plugins/maven-invoker-plugin/index.html)

[JaCoCo ](https://www.jacoco.org/jacoco/) is the defacto standard for test coverage measurement in the Java ecosystem.
For the unit tests, the JaCoCo  configuration was not a challenge.
How this configuration looks like can be read in the [post "Test Coverage Reports For Maven Projects In SonarQube 8.3.x"](/2020/06/10/test-coverage-reports-for-maven-projects-in-sonarqube-8-3-x).

The challenge was how to integrate JaCoCo into the invoker tests.
When running invoker tests, Maven starts for each invoker test an own Maven process.
So the question is how to integrate the JaCoCo Agent for the measurement in each new Maven process.
I cheated in the invoker tests of the project "JaCoCo Maven Plugin" how they integrate [JaCoCo into the Maven Invoker Plugin](https://github.com/jacoco/jacoco/blob/ec70eb541073805299c96f94c6d37d3778e8e4c8/jacoco-maven-plugin.test/pom.xml#L56).

## JaCoCo Configuration in Maven Invoker Plugin

First of all, it is required, that the JaCoCo Maven Plugin is configured with its goal `prepare-agent` in your POM.
Then the JaCoCo agent binary has to download into the local m2 repository of each invoker test.
For this use case, the Maven Invoker Plugin has the configuration parameter *extraArguments* in its goal *invoker:install*.
This parameter defines extra dependencies, that need to be installed on the local repository in addition to the dependencies that are defined in the test POM.
In our case, the JaCoCo agent.

```xml
<configuration>
  <extraArtifacts>
    <extraArtifact>org.jacoco:org.jacoco.agent:${jacoco.version}:jar:runtime</extraArtifact>
  </extraArtifacts>
</configuration>
```

In the last step, the JaCoCo Agent has to configured. Therefore, we use the JaCoCo Maven Plugin property `jacoco.propertyName`.
In our case, the JaCoCo Agent should use the Maven Opts defined by the invoker plugin.
If you want to limit the Java package for the test coverage measurement, you can use the JaCoCo Maven Plugin property `jacoco.includes`.

```xml
<properties>
   <!-- Enable recording of coverage during execution of maven-invoker-plugin -->
   <jacoco.propertyName>invoker.mavenOpts</jacoco.propertyName>
   <jacoco.includes>org.reficio.p2.*</jacoco.includes>
</properties>
```

Now JaCoCo creates a measurement of the coverage based on unit tests AND based on the integration tests.

The whole JaCoCo configuration can be found in the [P2 Maven Plugin POM](https://github.com/reficio/p2-maven-plugin/blob/master/pom.xml).
