---
title: 'Maven Project Setup for Mixing Spock 1.x and JUnit 5 Tests'
date: 2019-03-21
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
  - Groovy
  - Java
  - JUnit
  - JVM
  - Maven
  - Quality Assurance
  - Testing
tags:
  - groovy
  - junit5
  - maven
  - spock1.x

# comment: false # Disable comment if false.
---
I create a sample Groovy project for Maven, that mixes Spock tests and JUnit 5 tests in one project. In the next section I'll describe how to set up such kind of Maven project.

### Enable Groovy in the Project

First at all, you have to enable Groovy in your project. One possibility is to add the [GMavenPlus Plugin](https://groovy.github.io/GMavenPlus/) to your project.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.gmavenplus</groupId>
            <artifactId>gmavenplus-plugin</artifactId>
            <version>1.6.2</version>
            <executions>
                <execution>
                    <goals>
                        <goal>addSources</goal>
                        <goal>addTestSources</goal>
                        <goal>compile</goal>
                        <goal>compileTests</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

The goals `addSources` and `addTestSources` add Groovy (test) sources to Maven's main (test) sources. The default locations are `src/main/groovy` (for main source) and `src/test/groovy` (for test source). Goals `compile` and `compileTests` compile the Groovy (test) code. If you don't have Groovy main code, you can omit `addSource` and `compile`.

This above configuration is always using the latest released Groovy version. If you want to ensure that a specific Groovy version is used, you have to add the specific Groovy dependency to your classpath.

```xml
   <dependencies>
        <dependency>
            <groupId>org.codehaus.groovy</groupId>
            <artifactId>groovy</artifactId>
            <version>2.5.6</version>
        </dependency>
  </dependencies>
```

### Enable JUnit 5 in the Project

The simplest setup for using JUnit 5 in your project is to add the JUnit Jupiter dependency in your test class path and to configure the correct version of Maven Surefire Plugin (at least version 2.22.0).

```xml
    <dependencies>
<!--... maybe more dependencies -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.junit</groupId>
                <artifactId>junit-bom</artifactId>
                <version>${junit.jupiter.version}</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <build>
        <plugins>
        <!-- other plugins -->
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.1</version>
            </plugin>
        </plugins>
    </build>
```

### Enable Spock in the Project

Choosing the right Spock dependency depends on which Groovy version you are using in the project. In our case, a Groovy version 2.5. So we need Spock in version 1.x-groovy-2.5 in our test class path.

```xml
    <dependencies>
        <!-- more dependencies -->
        <dependency>
            <groupId>org.spockframework</groupId>
            <artifactId>spock-core</artifactId>
            <version>1.3-groovy-2.5</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

Now the expectation is that the Spock tests and the JUnit5 tests are executed in the Maven build. But only the JUnit5 tests are executed by Maven. So what happened?

I started to change the Maven Surefire Plugin version to 2.21.0. Then the Spock tests were executed, but no JUnit5 tests. The reason is that in the version 2.22.0 of Maven Surefire Plugin JUnit4 provider is replaced by JUnit Platform Provider as default. But Spock in version 1.x is based on JUnit4. This will be changed in Spock version 2. This version will be based on the JUnit5 Platform. Thus, for Spock 1.x, we have to add JUnit Vintage dependency to our test class path.

```xml
    <dependencies>
        <!-- more dependencies -->
          <dependency>  <!--Only necessary for surefire to run spock tests during the maven build -->
            <groupId>org.junit.vintage</groupId>
            <artifactId>junit-vintage-engine</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

This allows running elder JUnit (3/4) tests on the JUnit Platform. With this configuration both, Spock and JUnit 5 tests, are executed in the Maven build.

### Links

*   [Sample Maven Setup For Groovy including JUnit 5 and Spock on Github](https://github.com/sparsick/junit5-example/tree/master/junit5-spock1x)
*   [Maven GMaven Plus Plugin](https://groovy.github.io/GMavenPlus/)
*   [Maven Surefire Plugin - Using JUnit 5 Platform](https://maven.apache.org/surefire/maven-surefire-plugin/examples/junit-platform.html)
*   [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
*   [Spock Framework](https://github.com/spockframework/spock)
