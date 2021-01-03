---
title: 'Using JUnit 5 In Pre-Java 8 Projects'
date: 2019-01-20
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
  - JUnit
  - JVM
  - Maven
  - Testing
tags:
  - gradle
  - intellij
  - java8
  - junit5
  - maven
  - netbeans


# comment: false # Disable comment if false.
---
This post demonstrates how JUnit 5 can be used in pre-Java 8 projects and explains why it could be a good idea. JUnit 5 requires at least Java 8 as runtime environment, so you want to update your whole project to Java 8. But sometimes there exists reason why you can't immediately update your project to Java 8. For example, the version of your application server in production only supports Java 7. But an update isn't be taken quickly because of some issues in your production code. Now, the question is how can you use JUnit 5 without update your production code to Java 8? You can set up the Java version separately for production code and for test code.

```xml
<!-- in Maven -->
<build>
  <plugins>
    <plugin>
      <artifactId>maven-compiler-plugin</artifactId>
      <configuration>
        <source>7</source>
        <target>7</target>
        <testSource>8</testSource>
        <testTarget>8</testTarget>
      </configuration>
    </plugin>
  </plugins>
</build>
```
```groovy
// in Gradle
sourceCompatibility = '7'
targetCompatibility = '7'

compileTestJava {
  sourceCompatibility ='8'
  targetCompatibility = '8'
}
```

Precondition is that you use a Java 8 JDK for your build. If you try to use Java 8 feature in your Java 7 production code, Maven and Gradle will fail the build.

```shell
// Maven
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.8.0:compile (default-compile) on project junit5-in-pre-java8-projects: Compilation failure
[ERROR] /home/sparsick/dev/workspace/junit5-example/junit5-in-pre-java8-projects/src/main/java/Java7Class.java:[8,58] lambda expressions are not supported in -source 7
[ERROR]   (use -source 8 or higher to enable lambda expressions)
```
```shell
// Gradle
> Task :compileJava FAILED
/home/sparsick/dev/workspace/junit5-example/junit5-in-pre-java8-projects/src/main/java/Java7Class.java:8: error: lambda expressions are not supported in -source 7
Function<String, String > java8Feature = (input) -> input;
^
(use -source 8 or higher to enable lambda expressions)
1 error

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':compileJava'.
> Compilation failed; see the compiler error output for details.
```

Now you can introduce JUnit 5 in your project and start writing test with JUnit 5.

```xml
<!-- Maven-->
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter-api</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter-engine</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter-params</artifactId>
  <scope>test</scope>
</dependency>
<!-- junit-vintage-engine is needed for running elder JUnit4 test with JUnit5-->
<dependency>
  <groupId>org.junit.vintage</groupId>
  <artifactId>junit-vintage-engine</artifactId>
  <scope>test</scope>
</dependency>
```

```groovy
// in Gradle
dependencies {
  testCompile 'org.junit.jupiter:junit-jupiter-api:5.3.2'
  testCompile 'org.junit.jupiter:junit-jupiter-engine:5.3.2'
  testCompile 'org.junit.jupiter:junit-jupiter-params:5.3.2'
  testCompile 'org.junit.vintage:junit-vintage-engine:5.3.2'
}
```

Your old JUnit 4 tests need not be migrated, because JUnit 5 has a test engine, that can run JUnit 4 tests with JUnit 5. So use JUnit 5 for new tests and only migrate JUnit 4 tests if you have to touch them anyway.

Although you can't update your production code to a newer Java version, it has some benefit to update your test code to a newer one.

The biggest benefit is that you can start learning new language feature during your daily work when you write tests. You don't make the beginner's mistake in the production code. You have access to new tools that can help improve your tests. For example, in JUnit 5 it's more comfortable to write parameterized tests than in JUnit 4. In my experience, developer writes rather parameterized test with JUnit 5 than with JUnit 4 in a situation where parameterized test make sense.

The above described technique also works for other Java version. For example, your production code is on Java 11 and you want to use Java 12 feature in your test code. Another use case for this technique could be learning another JVM language like Groovy, Kotlin or Clojure in your daily work. Then use the new language in your test code.

For Maven projects, this approach has one little pitfall. IntelliJ IDEA ignores the Java version configuration for test code. It uses the configured Java version in production code section for the whole project. An [issue](https://youtrack.jetbrains.com/issue/IDEA-85478) is already opened. A workaround is also described in this issue. So only the Maven build gives you the feedback if your production code uses correct Java version. IntelliJ hasn't this problem for Gradle projects. Here, it uses the Java version just like it is configured in Gradle build file.

 The situation in Netbeans looks better for Maven projects. Netbeans loads the Java configuration for Maven project, correctly. For Gradle projects, I couldn't check it, because in Netbeans 10, there doesn't yet exist a Gradle plugin (status: January 2019, but for Netbeans 9, so maybe something will come)

### Links

*   [Example for a Project Set Up for Maven and Gradle](https://github.com/sparsick/junit5-example/tree/master/junit5-in-pre-java8-projects)
*   [IntelliJ's Maven issue](https://youtrack.jetbrains.com/issue/IDEA-85478)
*   [Gradle Plugin for Netbeans](http://plugins.netbeans.org/plugin/44510/gradle-support)
