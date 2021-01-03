---
title: 'Using Testcontainers in Spring Boot Tests For Database Integration Tests'
date: 2020-05-21
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
  - Quality Assurance
  - Spring Boot
  - Testcontainers
tags:
  - docker
  - hibernate
  - jdbc
  - jpa
  - junit5
  - spring
  - spring-data
  - testcontainers

# comment: false # Disable comment if false.
---
In this blog post I'd like to demonstrate how I integrate Testcontainers in Spring Boot tests for running integration tests with a database. I'm not using Testcontainers' Spring Boot modules. How it works with them, I will show in a separate blog post. All samples can be found on [GitHub](https://github.com/sparsick/testcontainers-spring-boot).

Why Testcontainers?
-------------------

Testcontainers is a library that helps to integrate infrastructure components like database in integration tests based on Docker Container. It helps to avoid writing integrated tests. These are kind of tests that will pass or fail based on the correctness of another system. With Testcontainers I have the control over these dependent systems.

Introducing the domain
----------------------

The further samples shows different approach how to save some hero objects through different repository implementations in a database and how the corresponding tests could look like.

```java
package com.github.sparsick.testcontainerspringboot.hero.universum;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import java.util.Objects;

public class Hero {
    private Long id;
    private String name;
    private String city;
    private ComicUniversum universum;

    public Hero(String name, String city, ComicUniversum universum) {
        this.name = name;
        this.city = city;
        this.universum = universum;
    }

    public String getName() {
        return name;
    }

    public String getCity() {
        return city;
    }

    public ComicUniversum getUniversum() {
        return universum;
    }
}

```

All further repositories are parts of a Spring Boot web application. So at the end of this blog post I will demonstrate how to write a test for the whole web application including a database. Let's start with an easy sample, a repository based on JDBC.

Testing Repository Based on JDBC
--------------------------------

Assume we have following repository implementation based on JDBC. We have two methods, one for adding a hero into the database and one for getting all heroes from the database.

```java
package com.github.sparsick.testcontainerspringboot.hero.universum;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import javax.sql.DataSource;
import java.util.Collection;

@Repository
public class HeroClassicJDBCRepository {

    private final JdbcTemplate jdbcTemplate;

    public HeroClassicJDBCRepository(DataSource dataSource) {
        jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public void addHero(Hero hero) {
        jdbcTemplate.update("insert into hero (city, name, universum) values (?,?,?)",
                hero.getCity(), hero.getName(), hero.getUniversum().name());

    }

    public Collection<Hero> allHeros() {
        return jdbcTemplate.query("select * From hero",
                (resultSet, i) -> new Hero(resultSet.getString("name"),
                                            resultSet.getString("city"),
                                            ComicUniversum.valueOf(resultSet.getString("universum"))));
    }

}
```

For this repository, we can write a normal JUnit5 tests without Spring application context loading. So first at all, we have to set up the dependencies to the test libraries, in this case, JUnit5 and Testcontainers. As build tool, I use Maven. Both test libraries provide so called [BOM "bill of material"](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Importing_Dependencies), that helps to avoid a version mismatch in my used dependencies. As database, I want to use MySQL. Therefore, I use the Testcontainers' module `mysql` additional to the core module `testcontainers`. It provides a predefined MySQL container. For simplifying the container setup specifically in JUnit5 test code, Testcontainers provides a JUnit5 module `junit-jupiter`.

```xml
    <dependencies>
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
        </dependencies>
    </dependencyManagement>
```

Now, we have everything to write the first test.

```java
package com.github.sparsick.testcontainerspringboot.hero.universum;

import ...

@Testcontainers
class HeroClassicJDBCRepositoryIT {
    @Container
    private MySQLContainer database = new MySQLContainer();

    private HeroClassicJDBCRepository repositoryUnderTest;

    @Test
    void testInteractionWithDatabase() {
        ScriptUtils.runInitScript(new JdbcDatabaseDelegate(database, ""),"ddl.sql");
        repositoryUnderTest = new HeroClassicJDBCRepository(dataSource());

        repositoryUnderTest.addHero(new Hero("Batman", "Gotham City", ComicUniversum.DC_COMICS));

        Collection<Hero> heroes = repositoryUnderTest.allHeros();

        assertThat(heroes).hasSize(1);
    }

    @NotNull
    private DataSource dataSource() {
        MysqlDataSource dataSource = new MysqlDataSource();
        dataSource.setUrl(database.getJdbcUrl());
        dataSource.setUser(database.getUsername());
        dataSource.setPassword(database.getPassword());
        return dataSource;
    }
}
```

Let's have a look how the database is prepared for the test. Firstly, we annotate the test class with `@Testcontainers`. Behind this annotation hides a JUnit5 extension provided by Testcontainers. It checks if Docker is installed on the machine, starts and stops the container during the test. But how Testcontainers knows which container it should start? Here, the annotation `@Container` helps. It marks container that should manage by the Testcontainers extension. In this case, a `MySQLContainer` provided by Testcontainers module `mysql`. This class provides a MySQL Docker container and handles such things like setting up database user, recognizing when the database is ready to use etc. As soon as the database is ready to use, the database schema has to be set up. Testcontainers can also provide support here. `ScriptUtils._runInitScript_(new JdbcDatabaseDelegate(database, ""),"ddl.sql");` ensure that the schema is set up like it defines in SQL script `ddl.sql`.

```sql
-- ddl.sql
create table hero (id bigint AUTO_INCREMENT PRIMARY KEY, city varchar(255), name varchar(255), universum varchar(255)) engine=InnoDB
```

Now we are ready to set up our repository under test. Therefore, we need the database connection information for the `DataSource` object. Under the hood, Testcontainers searches after an available port and bind the container on this free port. This port number is different on every container start via Testcontainers. Furthermore, it configures the database in container with a user and password. Therefore, we have to ask the `MySQLContainer` object how the database credentials and the JDBC URL are. With this information, we can set up the repository under test (`repositoryUnderTest = new HeroClassicJDBCRepository(dataSource());`) and finish the test.

If you run the test and you get the following error message:

```shell
17:18:50.990 [ducttape-1] DEBUG com.github.dockerjava.core.command.AbstrDockerCmd - Cmd: org.testcontainers.dockerclient.transport.okhttp.OkHttpDockerCmdExecFactory$1@1adc57a8
17:18:51.492 [ducttape-1] DEBUG org.testcontainers.dockerclient.DockerClientProviderStrategy - Pinging docker daemon...
17:18:51.493 [ducttape-1] DEBUG com.github.dockerjava.core.command.AbstrDockerCmd - Cmd: org.testcontainers.dockerclient.transport.okhttp.OkHttpDockerCmdExecFactory$1@3e5b3a3b
17:18:51.838 [main] DEBUG org.testcontainers.dockerclient.DockerClientProviderStrategy - UnixSocketClientProviderStrategy: failed with exception InvalidConfigurationException (ping failed). Root cause LastErrorException ([111] Verbindungsaufbau abgelehnt)
17:18:51.851 [main] DEBUG org.rnorth.tcpunixsocketproxy.ProxyPump - Listening on localhost/127.0.0.1:41039 and proxying to /var/run/docker.sock
17:18:51.996 [ducttape-0] DEBUG org.testcontainers.dockerclient.DockerClientProviderStrategy - Pinging docker daemon...
17:18:51.997 [ducttape-1] DEBUG org.testcontainers.dockerclient.DockerClientProviderStrategy - Pinging docker daemon...
17:18:51.997 [ducttape-0] DEBUG com.github.dockerjava.core.command.AbstrDockerCmd - Cmd: org.testcontainers.dockerclient.transport.okhttp.OkHttpDockerCmdExecFactory$1@5d43d23e
17:18:51.997 [ducttape-1] DEBUG com.github.dockerjava.core.command.AbstrDockerCmd - Cmd: org.testcontainers.dockerclient.transport.okhttp.OkHttpDockerCmdExecFactory$1@7abf08d2
17:18:52.002 [tcp-unix-proxy-accept-thread] DEBUG org.rnorth.tcpunixsocketproxy.ProxyPump - Accepting incoming connection from /127.0.0.1:41998
17:19:01.866 [main] DEBUG org.testcontainers.dockerclient.DockerClientProviderStrategy - ProxiedUnixSocketClientProviderStrategy: failed with exception InvalidConfigurationException (ping failed). Root cause TimeoutException (null)
17:19:01.870 [main] ERROR org.testcontainers.dockerclient.DockerClientProviderStrategy - Could not find a valid Docker environment. Please check configuration. Attempted configurations were:
17:19:01.872 [main] ERROR org.testcontainers.dockerclient.DockerClientProviderStrategy -     EnvironmentAndSystemPropertyClientProviderStrategy: failed with exception InvalidConfigurationException (ping failed)
17:19:01.873 [main] ERROR org.testcontainers.dockerclient.DockerClientProviderStrategy -     EnvironmentAndSystemPropertyClientProviderStrategy: failed with exception InvalidConfigurationException (ping failed)
17:19:01.874 [main] ERROR org.testcontainers.dockerclient.DockerClientProviderStrategy -     UnixSocketClientProviderStrategy: failed with exception InvalidConfigurationException (ping failed). Root cause LastErrorException ([111] Verbindungsaufbau abgelehnt)
17:19:01.875 [main] ERROR org.testcontainers.dockerclient.DockerClientProviderStrategy -     ProxiedUnixSocketClientProviderStrategy: failed with exception InvalidConfigurationException (ping failed). Root cause TimeoutException (null)
17:19:01.875 [main] ERROR org.testcontainers.dockerclient.DockerClientProviderStrategy - As no valid configuration was found, execution cannot continue
17:19:01.900 [main] DEBUG 🐳 [mysql:5.7.22] - mysql:5.7.22 is not in image name cache, updating...
Mai 01, 2020 5:19:01 NACHM. org.junit.jupiter.engine.execution.JupiterEngineExecutionContext close
SEVERE: Caught exception while closing extension context: org.junit.jupiter.engine.descriptor.MethodExtensionContext@2e6a5539
org.testcontainers.containers.ContainerLaunchException: Container startup failed
	at org.testcontainers.containers.GenericContainer.doStart(GenericContainer.java:322)
	at org.testcontainers.containers.GenericContainer.start(GenericContainer.java:302)
	at org.testcontainers.junit.jupiter.TestcontainersExtension$StoreAdapter.start(TestcontainersExtension.java:173)
	at org.testcontainers.junit.jupiter.TestcontainersExtension$StoreAdapter.access$100(TestcontainersExtension.java:160)
	at org.testcontainers.junit.jupiter.TestcontainersExtension.lambda$null$3(TestcontainersExtension.java:50)
	at org.junit.jupiter.engine.execution.ExtensionValuesStore.lambda$getOrComputeIfAbsent$0(ExtensionValuesStore.java:81)
	at org.junit.jupiter.engine.execution.ExtensionValuesStore$MemoizingSupplier.get(ExtensionValuesStore.java:182)
	at org.junit.jupiter.engine.execution.ExtensionValuesStore.closeAllStoredCloseableValues(ExtensionValuesStore.java:58)
	at org.junit.jupiter.engine.descriptor.AbstractExtensionContext.close(AbstractExtensionContext.java:73)
	at org.junit.jupiter.engine.execution.JupiterEngineExecutionContext.close(JupiterEngineExecutionContext.java:53)
	at org.junit.jupiter.engine.descriptor.JupiterTestDescriptor.cleanUp(JupiterTestDescriptor.java:222)
	at org.junit.jupiter.engine.descriptor.JupiterTestDescriptor.cleanUp(JupiterTestDescriptor.java:57)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$cleanUp$9(NodeTestTask.java:151)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.cleanUp(NodeTestTask.java:151)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:83)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1540)
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:38)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$5(NodeTestTask.java:139)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$7(NodeTestTask.java:125)
	at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:135)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:123)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:122)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:80)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1540)
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:38)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$5(NodeTestTask.java:139)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$7(NodeTestTask.java:125)
	at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:135)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:123)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:122)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:80)
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.submit(SameThreadHierarchicalTestExecutorService.java:32)
	at org.junit.platform.engine.support.hierarchical.HierarchicalTestExecutor.execute(HierarchicalTestExecutor.java:57)
	at org.junit.platform.engine.support.hierarchical.HierarchicalTestEngine.execute(HierarchicalTestEngine.java:51)
	at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:229)
	at org.junit.platform.launcher.core.DefaultLauncher.lambda$execute$6(DefaultLauncher.java:197)
	at org.junit.platform.launcher.core.DefaultLauncher.withInterceptedStreams(DefaultLauncher.java:211)
	at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:191)
	at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:128)
	at com.intellij.junit5.JUnit5IdeaTestRunner.startRunnerWithArgs(JUnit5IdeaTestRunner.java:69)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:230)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:58)
Caused by: org.testcontainers.containers.ContainerFetchException: Can't get Docker image: RemoteDockerImage(imageNameFuture=java.util.concurrent.CompletableFuture@539d019[Completed normally], imagePullPolicy=DefaultPullPolicy(), dockerClient=LazyDockerClient.INSTANCE)
	at org.testcontainers.containers.GenericContainer.getDockerImageName(GenericContainer.java:1265)
	at org.testcontainers.containers.GenericContainer.logger(GenericContainer.java:600)
	at org.testcontainers.containers.GenericContainer.doStart(GenericContainer.java:311)
	... 47 more
Caused by: java.lang.IllegalStateException: Previous attempts to find a Docker environment failed. Will not retry. Please see logs and check configuration
	at org.testcontainers.dockerclient.DockerClientProviderStrategy.getFirstValidStrategy(DockerClientProviderStrategy.java:78)
	at org.testcontainers.DockerClientFactory.client(DockerClientFactory.java:115)
	at org.testcontainers.LazyDockerClient.getDockerClient(LazyDockerClient.java:14)
	at org.testcontainers.LazyDockerClient.inspectImageCmd(LazyDockerClient.java:12)
	at org.testcontainers.images.LocalImagesCache.refreshCache(LocalImagesCache.java:42)
	at org.testcontainers.images.AbstractImagePullPolicy.shouldPull(AbstractImagePullPolicy.java:24)
	at org.testcontainers.images.RemoteDockerImage.resolve(RemoteDockerImage.java:62)
	at org.testcontainers.images.RemoteDockerImage.resolve(RemoteDockerImage.java:25)
	at org.testcontainers.utility.LazyFuture.getResolvedValue(LazyFuture.java:20)
	at org.testcontainers.utility.LazyFuture.get(LazyFuture.java:27)
	at org.testcontainers.containers.GenericContainer.getDockerImageName(GenericContainer.java:1263)
	... 49 more



org.testcontainers.containers.ContainerLaunchException: Container startup failed
```

This error message means that the Docker daemon is not running. After ensuring that Docker daemon is running, the test run is successful.

There are very many debug messages in the console output. The logging output in tests can be configured by a `logback.xml` file in `src/test/resources`:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
    <root level="info">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

[Spring Boot documentation about logging](https://docs.spring.io/spring-boot/docs/2.3.x/reference/htmlsingle/#boot-features-custom-log-configuration) recommends to use `logback-spring.xml` as configuration file. But normal JUnit5 tests don't recognize it, only `@SpringBootTest` annotated tests. `logback.xml` is used by both kind of tests.

Testing Repository based on JPA Entity Manager
----------------------------------------------

Now, we want to implement a repository based on JPA with a classic entity manager. Assume, we have following implementation with three methods, adding heroes to the database, finding heroes by search criteria and getting all heroes from the database. The entity manager is configured by Spring's application context (`@PersistenceContext` is responsible for that).

```java
package com.github.sparsick.testcontainerspringboot.hero.universum;

import ...

@Repository
public class HeroClassicJpaRepository {

    @PersistenceContext
    private EntityManager em;

    @Transactional
    public void addHero(Hero hero) {
        em.persist(hero);
    }

    public Collection<Hero> allHeros() {
        return em.createQuery("Select hero FROM Hero hero", Hero.class).getResultList();
    }

    public Collection<Hero> findHerosBySearchCriteria(String searchCriteria) {
        return em.createQuery("SELECT hero FROM Hero hero " +
                        "where hero.city LIKE :searchCriteria OR " +
                        "hero.name LIKE :searchCriteria OR " +
                        "hero.universum = :searchCriteria",
                Hero.class)
                .setParameter("searchCriteria", searchCriteria).getResultList();
    }

}
```

As JPA implementation, we choose Hibernate and MySQL as database provider. We have to configure which dialect should Hibernate use.

```properties
# src/main/resources/application.properties
spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
```

In `application.properties` you also configure the database connection etc.

For setting up the entity manager in a test correctly, we have to run the test with an application context, so that entity manager is configured correctly by Spring.

Spring Boot brings some test support classes. Therefore, we have to add a further test dependency to the project.

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
```

This starter also includes JUnit Jupiter dependency and dependencies from other test library, so you can remove these dependencies from your dependency declaration if you want to.

Now, we have everything for writing the test.

```java
package com.github.sparsick.testcontainerspringboot.hero.universum;

import ...

@SpringBootTest
@Testcontainers
class HeroClassicJpaRepositoryIT {
    @Container
    private static MySQLContainer database = new MySQLContainer();

    @Autowired
    private HeroClassicJpaRepository repositoryUnderTest;

    @Test
    void findHeroByCriteria(){
        repositoryUnderTest.addHero(new Hero("Batman", "Gotham City", ComicUniversum.DC_COMICS));

        Collection<Hero> heros = repositoryUnderTest.findHerosBySearchCriteria("Batman");

        assertThat(heros).contains(new Hero("Batman", "Gotham City", ComicUniversum.DC_COMICS));
    }

    @DynamicPropertySource
    static void databaseProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", database::getJdbcUrl);
        registry.add("spring.datasource.username", database::getUsername);
        registry.add("spring.datasource.password", database::getPassword);
    }
}
```

The test class is annotated with some annotations. The first one is @`SpringBootTest` thereby the Spring application context is started during the test. The last one is `@Testcontainers` . This annotation we already know from the last test. It is a JUnit5 extension that manage starting and stopping the docker container during the test. Like we see at above JDBC test, we annotate database container `private static MySQLContainer database = new MySQLContainer();` with `@Container`. It marks that this container should be managed by Testcontainers. Here is a little difference to above JDBC set up. Here, `MySQLContainer database` is `static` and in the JDBC set up it is a normal class field. Here, it has to be static because the container has to start before the application context starts, so that we have a change to pass the database connection configuration to the application context. For this, `static method databaseProperties` is responsible. Here, it is important that this method is annotated by `@DynamicPropertySource`. It overrides the application context configuration during the start phase. In our case, we want to override the database connection configuration with the database information that we get from database container object managed by Testcontainers. The last step is to set up the database schema in the database. Here JPA can help. It can create a database schema automatically. You have to configure it with

```proeprties
# src/test/resources/application.properties
spring.jpa.hibernate.ddl-auto=update
```

Now, we can inject the repository into the test (`@Autowired private HeroClassicJpaRepository repositoryUnderTest`). This repository is configured by Spring and ready to test.

Testing Repository based on Spring Data JPA
-------------------------------------------

Today, it is common in a Spring Boot application to use JPA in combination with Spring Data, so we rewrite our repository to use Spring Data JPA instead of plain JPA. The result is an interface that extends Spring Data's `CrudRepository`, so we have all basic operation like save, delete, update find by id etc. . For searching by criteria functionality, we have to define a method with `@Query` annotation that have a JPA query.

```java
package com.github.sparsick.testcontainerspringboot.hero.universum;

import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.CrudRepository;
import org.springframework.data.repository.query.Param;

import java.util.List;

public interface HeroSpringDataJpaRepository extends CrudRepository<Hero, Long> {

    @Query("SELECT hero FROM Hero hero where hero.city LIKE :searchCriteria OR hero.name LIKE :searchCriteria OR hero.universum = :searchCriteria")
    List<Hero> findHerosBySearchCriteria(@Param("searchCriteria") String searchCriteria);
}

```

As mentioned above in classic JPA sample so also here, we have to configure which SQL dialect our chosen JPA implementation Hibernate should use and how the database schema should set up.

The same with the test configuration, again we need a test with a Spring application context to configure the repository correctly for the test. But here we don't need to start the whole application context with `@SpringBootTest`. Instead, we use `@DataJpaTest`. This annotation starts an application context only with beans that are needed for the persistence layer.

```java
package com.github.sparsick.testcontainerspringboot.hero.universum;

import ...

@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class HeroSpringDataJpaRepositoryIT {
    @Container
    private static MySQLContainer database = new MySQLContainer();

    @Autowired
    private HeroSpringDataJpaRepository repositoryUnderTest;

    @Test
    void findHerosBySearchCriteria() {
        repositoryUnderTest.save(new Hero("Batman", "Gotham City", ComicUniversum.DC_COMICS));

        Collection<Hero> heros = repositoryUnderTest.findHerosBySearchCriteria("Batman");

        assertThat(heros).hasSize(1).contains(new Hero("Batman", "Gotham City", ComicUniversum.DC_COMICS));
    }

    @DynamicPropertySource
    static void databaseProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url",database::getJdbcUrl);
        registry.add("spring.datasource.username", database::getUsername);
        registry.add("spring.datasource.password", database::getPassword);
    }
}
```

`@DataJpaTest` starts an in-memory database as default. But we want that a containerized database is used, provided by Testcontainers. Therefore, we have to add the annotation `@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)`. This disables starting an in-memory database. The remaining test configuration is the same as the configuration in above test for the plain JPA example.

Testing Repositories but Reusing a database
-------------------------------------------

With the increasing number of tests, it becomes more and more important that each test takes quite a long time, because each time a new database is started and initialized. One idea is to reuse the database in each test. Here the [Single Container Pattern](https://www.testcontainers.org/test_framework_integration/manual_lifecycle_control/#singleton-containers) can help. A database is started and initialized once before all tests start running. For that, each test that need a database has to extend an abstract class, that is responsible for starting and initializing a database once before all tests run.

```java
package com.github.sparsick.testcontainerspringboot.hero.universum;

import ...


public abstract class DatabaseBaseTest {
    static final MySQLContainer DATABASE = new MySQLContainer();

    static {
        DATABASE.start();
    }

    @DynamicPropertySource
    static void databaseProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", DATABASE::getJdbcUrl);
        registry.add("spring.datasource.username", DATABASE::getUsername);
        registry.add("spring.datasource.password", DATABASE::getPassword);
    }
}
```

In this abstract class we configure the database that is started once for all tests that extend this abstract class and the application context with that database. Please note, that we don't use Testcontainers' annotations here, because this annotation takes care that the container is started and stopped after each test. But this we would avoid. Therefore, we start the database by ourselves. For stopping database we don't need to take care. For this Testcontainers' side-car container ryuk takes care.

Now, each test class, that need a database, extends this abstract class. The only thing, that we have to configure, is how the application context should be initialized. That means, when you need the whole application context then use `@SpringBootTest`. When you need only persistence layer then use `@DataJpaTest` with `@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)`.

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class HeroSpringDataJpaRepositoryReuseDatabaseIT extends DatabaseBaseTest {

    @Autowired
    private HeroSpringDataJpaRepository repositoryUnderTest;

    @Test
    void findHerosBySearchCriteria() {
        repositoryUnderTest.save(new Hero("Batman", "Gotham City", ComicUniversum.DC_COMICS));

        Collection<Hero> heros = repositoryUnderTest.findHerosBySearchCriteria("Batman");

        assertThat(heros).contains(new Hero("Batman", "Gotham City", ComicUniversum.DC_COMICS));
    }
}
```

Testing the whole Web Application including Database
----------------------------------------------------

Now we want to test our whole application, from controller to database. The controller implementation looks like this:

```java
@RestController
public class HeroRestController {

    private final HeroSpringDataJpaRepository heroRepository;

    public HeroRestController(HeroSpringDataJpaRepository heroRepository) {
        this.heroRepository = heroRepository;
    }

    @GetMapping("heros")
    public Iterable<Hero> allHeros(String searchCriteria) {
        if (searchCriteria == null || searchCriteria.equals("")) {
            return heroRepository.findAll();

        }
        return heroRepository.findHerosBySearchCriteria(searchCriteria);
    }

    @PostMapping("hero")
    public void hero(@RequestBody Hero hero) {
        heroRepository.save(hero);
    }
}
```

The test class that test the whole way from database to controller looks like that

```java
@SpringBootTest
@AutoConfigureMockMvc
@Testcontainers
class HeroRestControllerIT {

    @Container
    private static MySQLContainer database = new MySQLContainer();

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private HeroSpringDataJpaRepository heroRepository;

    @Test
    void allHeros() throws Exception {
        heroRepository.save(new Hero("Batman", "Gotham City", ComicUniversum.DC_COMICS));
        heroRepository.save(new Hero("Superman", "Metropolis", ComicUniversum.DC_COMICS));

        mockMvc.perform(get("/heros"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$[*].name", containsInAnyOrder("Batman", "Superman")));
    }

    @DynamicPropertySource
    static void databaseProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", database::getJdbcUrl);
        registry.add("spring.datasource.username", database::getUsername);
        registry.add("spring.datasource.password", database::getPassword);
    }
}
```

The test set up for the database and the application is known by the test from the above sections. One thing is different. We add MockMVC support with `@AutoConfigureMockMvc`. This helps to write tests through the HTTP layer.

Of course, you can also use the single container pattern in which the abstract class `DatabaseBaseTest` is extended.

Conclusion and Overview
-----------------------

This blog post shows how we can write tests for some persistence layer implementations in Spring Boot with Testcontainers. We also see how to reuse database instance for several tests and how to write test for the whole web application from controller tor database. All code snippet can be found on [GitHub](https://github.com/sparsick/testcontainers-spring-boot). In a further blog post I will show how to write test with Testcontainers Spring Boot modules.

Do you have other ideas for writing tests for persistence layer? Please let me know and write a comment.

Further Information
-------------------

1.  [Concept of BOM "bill of material"](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Importing_Dependencies)
2.  [Testcontainers](https://www.testcontainers.org/)
3.  [Spring Boot Documentation - Logging](https://docs.spring.io/spring-boot/docs/2.3.x/reference/htmlsingle/#boot-features-custom-log-configuration)
4.  [Spring Boot Documentation - Auto-configured Data JPA Tests](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jpa-test)
5.  [Testcontainers - Single Container Pattern](https://www.testcontainers.org/test_framework_integration/manual_lifecycle_control/#singleton-containers)
6.  [Spring Boot Documentation - MockMVC](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-testing-spring-boot-applications-testing-with-mock-environment)
7.  [Full example in GitHub repository](https://github.com/sparsick/testcontainers-spring-boot)
