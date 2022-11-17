---
title: "Using Testcontainer in Spring Boot Tests combined with JUnit5 for Selenium Tests" # Title of the blog post.
date: 2022-11-02T10:01:42+01:00 # Date of post creation.
description: "Running Selennium Tests with Testcontainers and JUnit5 in Spring Boot Tests" # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
draft: true # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Java
  - JUnit
  - JVM
  - Quality Assurance
  - Spring Boot
  - Testcontainers
tags:
  - docker
  - junit5
  - selenium
  - spring
  - testcontainers
# comment: false # Disable comment if false.
---

In this blog post I'd like to demonstrate how I integrate Testcontainers in Spring Boot tests for running UI tests with Selenium. 
The sample project can be found on [GitHub](https://github.com/sparsick/testcontainers-spring-boot).


Why Testcontainers?
-------------------

Testcontainers is a library that helps to integrate infrastructure components like Selenium in integration tests based on Docker Container. 
It helps to avoid writing integrated tests. 
These are kind of tests that will pass or fail based on the correctness of another system. 
In my case, Selenium. 
With Testcontainers I have the control over this dependent system. 

If you want to learn how to write integration tests with database integration, please have a look on my blog post about [Using Testcontainers in Spring Boot Tests For Database Integration Tests](https://blog.sandra-parsick.de/2020/05/21/using-testcontainers-in-spring-boot-tests-for-database-integration-tests/)



Introducing the Subject Under Test
---------------------------------

The sample project is a simple web application with an UI based on Thymeleaf, Spring Boot and mysql database. 
In the next sections, I demonstrate how to write an End2End-test for the start page of the application.

![Startpage of Hero Web app]()

<!-- modell of the application -->


Preparing a Test with the Whole Application
----------------------

I want to test the whole application including database. 
Therefore, I need a JUnit5 test that load the complete Spring Boot context and a MySQL database.


The first step is to set up the dependencies to the test libraries.
In this case, JUnit5, Testcontainers and Spring Boot's test helper classes. 
As build tool, I use Maven. 
All test libraries provide so called [BOM "bill of material"](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Importing_Dependencies), that helps to avoid a version mismatch in my used dependencies. 
As database, I want to use MySQL. 
Therefore, I use the Testcontainers' module `mysql` additional to the core module `testcontainers`. 
It provides a predefined MySQL container. 
For simplifying the container setup specifically in JUnit5 test code, Testcontainers provides a JUnit5 module `junit-jupiter`.
For Spring Boot's test helper class, I need the test dependency `spring-boot-starter-test`.

```xml
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>testcontainers</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>mysql</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-test</artifactId>
          <scope>test</scope>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.junit</groupId>
                <artifactId>junit-bom</artifactId>
                <version>${junit.jupiter.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.testcontainers</groupId>
                <artifactId>testcontainers-bom</artifactId>
                <version>${testcontainers.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

Now, everything is prepared for setup the test.

```java
package com.github.sparsick.testcontainerspringboot.hero.universum;


import ...

@Testcontainers
@SpringBootTest
class HeroStartPageIT {
    @Container
    private static MySQLContainer database = new MySQLContainer("mysql:5.7.34");

    @Test
    void contextLoad() {
      
    }

    @DynamicPropertySource
    static void databaseProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url",database::getJdbcUrl);
        registry.add("spring.datasource.username", database::getUsername);
        registry.add("spring.datasource.password", database::getPassword);
    }


}

``` 

The test class is annotated with some annotations. 
The first one is `@SpringBootTest` thereby the Spring application context is started during the test. 
The last one is `@Testcontainers` . 
It is a JUnit5 extension provided by Testcontainers that manage starting and stopping the docker container during the test. 
It checks if Docker is installed on the machine, starts and stops the container during the test. 
But how Testcontainers knows which container it should start? Here, the annotation `@Container` helps. 
It marks container that should manage by the Testcontainers extension. 
In this case, a `MySQLContainer` provided by Testcontainers module `mysql`. 
This class provides a MySQL Docker container and handles such things like setting up database user, recognizing when the database is ready to use etc. 
As soon as the database is ready to use, the database schema has to be set up.
`MySQLContainer` database is static, because the container has to start before the application context starts, so that we have a change to pass the database connection configuration to the application context. 
For this, static method `databaseProperties` is responsible. 
Here, it is important that this method is annotated by `@DynamicPropertySource`. 
It overrides the application context configuration during the start phase. 
Here, I want to override the database connection configuration with the database information that I get from database container object managed by Testcontainers. 
The last step is to set up the database schema in the database. Here JPA can help. It can create a database schema automatically. I have to configure it with

```properties
# src/test/resources/application.properties
spring.jpa.hibernate.ddl-auto=update
```

