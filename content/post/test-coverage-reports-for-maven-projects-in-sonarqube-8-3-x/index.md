---
title: 'Test Coverage Reports For Maven Projects In SonarQube 8.3.x'
date: 2020-06-10
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
  - Java
  - Maven
  - Quality Assurance
  - SonarQube
  - Testing
tags:
  - integration-testing
  - jacoco
  - maven
  - sonarqube
  - test-coverage
  - test-report
  - unit-testing


# comment: false # Disable comment if false.
---
Some years ago I write a [blog post](https://blog.sandra-parsick.de/2015/01/30/unit-and-integration-test-reports-for-maven-projects-in-sonarqube/) about how to generate test reports in SonarQube separate in test report for unit tests and for integration tests. Since SonarQube 6.2 the test report isn't separate in these categories any more (see [SonarQube's blog post](https://blog.sonarsource.com/sonarqube-6-2-in-screenshots/)). SonarQube merges all test reports to one test report with an overall coverage. So how to configure JaCoCo Maven Plugin if you have separate your tests in unit tests (running by Maven Surefire Plugin) and integration tests (running by Maven Failsafe Plugin) in your Maven project.

In the following sections, a solution is presented that meets following criteria:

*   Maven is used as build tool.
*   The project can be a multi module project.
*   Unit tests and integration tests are parts of each module.
*   Test coverage is measured by JaCoCo Maven Plugin.

The road map for the next section is that firstly the Maven project structure is shown for the separation of unit and integration tests. Then the Maven project configuration is shown for having separate unit test runs and integration test runs.  After that, we have a look on the Maven project configuration for the test report generation that covers unit and integration tests. At the end, SonarQube's configuration is shown for the test report visualization in the SonarQube's dashboard.

Maven Project Structure
-----------------------

At first, we look at how a default Maven project structure looks like for a single module project.

```shell
my-app
├── pom.xml
├── src
│   ├── main
│   │   └── java
│   └── test
│       └── java
```

The directory `src/main/java` contains the production source code and the directory `src/test/java` contains the test source code. We could put unit tests and integration tests together in this directory. But we want to separate these two types of tests in separate directories. Therefore, we add a new directory called `src/it/java.` Then unit tests are put in the directory `src/test/java` and the integration tests are put in the directory `src/it/java,` so the new project structure looks like the following one.

```shell
my-app
├── pom.xml
├── src
│   ├── it
│   │   └── java
│   ├── main
│   │   └── java
│   └── test
│       └── java
```

Unit And Integration Test Runs
------------------------------

Fortunately, the unit test run configuration is a part of the Maven default project configuration. Maven runs these tests automatically if following criteria are met:

*   The tests are in the directory `src/test/java` and
*   the test class name either starts with `Test` or ends with `Test` or `TestCase.`

Maven runs these tests during the [Maven's build lifecylce](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html) phase `test.`

The integration test run configuration has to be done manually. It exists Maven plugins that can help. We want that the following criteria are met:

*   integration tests are stored in the directory `src/it/java` and
*   the integration test class name either starts `IT` or ends with `IT` or `ITCase` and
*   integrations tests runs during the Maven's build lifecycle phase `integration-test.`

Firstly, Maven has to know that it has to include the directory `src/it/java` to its test class path. Here, the Build Helper Maven Plugin can help. It adds the directory `src/it/java` to the test class path.

```xml
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>build-helper-maven-plugin</artifactId>
                <version>3.1.0</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>add-test-source</goal>
                            <goal>add-test-resource</goal>
                        </goals>
                        <configuration>
                            <sources>
                                <source>src/it/java</source>
                            </sources>
                            <resources>
                                <resource>
                                    <directory>src/it/resources</directory>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```

The above code snippet has to be inserted into the section `<project><build><plugins>` in the project root pom.

Maven's build lifecycle contains a phase called `integration-test.` In this phase, we want to run the integration test. Fortunately, Maven Failsafe Plugins' goal `integration-test` is bind to this phase automatically when it's set up in the POM. If you want that the build fails when the integration tests fails then the goal `verify` also has to be added into the POM:

```xml
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>3.0.0-M4</version>
                <configuration>
                    <encoding>${project.build.sourceEncoding}</encoding>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

Again, the above code snippet also has to be inserted into the section `<project><build><plugins>` in the project root pom. Then Maven Failsafe Plugin runs the integration tests automatically, when their class name either starts with `IT` or ends with `IT `or `ITCase.`

Test Report Generation
----------------------

We want to use the JaCoCo Maven Plugin for the test report generation. It should generate test reports for the unit tests and for the integration tests. Therefore, the plugin has to two separated agents, that have to be prepared. Then they generate the report during the test runs. The Maven's build lifecycle contains own phases for preparation before the test phases (`test` and `integration-test`). The preparation phase for the `test` phase is called `process-test-classes` and the preparation phase for `integration-test` phase is called `pre-integration-test`. JaCoCo binds its agent to these phases automatically, when its goals `prepare-agent` and `prepare-agent-integration` are set up in the POM. But this is not enough. JaCoCo also has to create a report, so that SonarQube can read the reports for the visualization. Therefore, we have to add the goals `report` and `report-integration` in the POM:

```xml
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.5</version>
                <executions>    
                    <execution>
                        <goals>  
                            <goal>prepare-agent</goal>
                            <goal>prepare-agent-integration</goal>
                            <goal>report</goal>
                            <goal>report-integration</goal>
                        </goals>  
                    </execution>
                </executions>  
            </plugin>
```

Again, it is a part of the section `<project><build><plugins>`.

Now, we can run the goal `mvn verify` and our project is built inclusive unit and integration test and inclusive generating two test reports.

SonarQube Test Report Visualization
-----------------------------------

Now, we want to visualize our test reports in SonarQube. Therefore, we have to run the Sonar Maven 3 Plugin (command `mvn sonar:sonar`)  in our project after a successful build. So Sonar Maven Plugin knows where to upload the report, we have to configure our SonarQube instance in `~/.m2/setting.xml`:

```xml
    <profile>
      <id>sonar</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <!-- Optional URL to server. Default value is http://localhost:9000 -->
        <sonar.host.url>http://localhost:9000</sonar.host.url>
      </properties>
    </profile>
```

When we open our project in the SonarQube dashboard, we see the overall test coverage report.

Summary
-------

This blog describes how to generate test reports for a Maven build if unit tests and integration tests run separately. On [GitHub](https://github.com/sparsick/sonarqube-test-report-with-maven/tree/sonarqube-8.3.1/sonarqube-test-report-with-maven), I host a sample project that demonstrate all configuration steps. As technical environment I use

*   Maven 3.6.3
*   Maven Plugins:
    *   Maven Surefire Plugin
    *   Maven Failsafe Plugin
    *   Build Helper Maven Plugin
    *   Jacoco Maven Plugin
    *   Sonar Maven Plugin
*   SonarQube 8.3.1
*   Java 11

Links
-----

1.  Jacoco Maven plugin [project site](http://www.eclemma.org/jacoco/trunk/doc/maven.html)
2.  Maven Failsafe Plugin  [project site](http://maven.apache.org/surefire/maven-failsafe-plugin/index.html)
3.  Build Helper Maven Plugin [project site](http://mojo.codehaus.org/build-helper-maven-plugin/)
4.  SonarQube documentation about [Test Coverage in common](http://docs.sonarqube.org/display/PLUG/Code+Coverage+by+Integration+Tests+for+Java+Project)
5.  A sample Maven project on [GitHub](https://github.com/sparsick/sonarqube-test-report-with-maven/tree/sonarqube-8.3.1/sonarqube-test-report-with-maven)
