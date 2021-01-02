---
title: 'Spring Web Application With Hessian Services As a Maven Project'
date: 2012-11-12
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
  - Maven
  - Spring Framework
tags:
  - hessian
  - maven
  - spring
  - tomcat


# comment: false # Disable comment if false.
---

This post describes how to set up a Maven project for a Spring web application with Hessian Service. It also shows how to set up the deployment for exposing the Hessian service and how to set up a client to consume the Hessian service. The Spring Framework version is 3.1.1.RELEASE and the Hessian version is 4.0.7.

### Maven Set Up For Server

Our Maven project has three modules

*   hello-world-api
*   hello-world-impl
*   hello-word-war

```xml
<modules>
  <module>hello-world-api</module>
  <module>hello-world-impl</module>
  <module>hello-world-war</module>
</modules>
```

The module `hello-world-api` contains the interfaces of the services that Hessian server and Hessian client need for the communication. The module `hello-world-impl` contains the implementation of the services that are deployed on the server side. The module `hello-world-war` contains the configuration for the servlet container and the configuration which services should be exported as a  Hessian service.

### The Service Interface Definition

The module `hello-world-api` should become a JAR artifact so the packaging `jar` for this Maven module:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.github.skosmalla.spring.hessian</groupId>
    <artifactId>hello-world-spring-hessian</artifactId>
    <version>1.0.0-SNAPSHOT</version>
  </parent>

  <artifactId>hello-world-api</artifactId>

  <name>Hello World Api</name>

</project>
```
The jar contains the interfaces of the services. An example

```java
package com.github.skosmalla.hello.world.spring.hessian;

public interface HelloWorld {

  public String welcome();

}
```
### The Service Implementation

The module `hello-world-impl` also becomes a JAR artifact. This jar contains the service implementation for the server. The service implementation could look like following code:

```java
package com.github.skosmalla.hello.world.spring.hessian;

public class HelloWorldImpl implements HelloWorld {

  @Override
  public String welcome() {
    return "Hello World";
  }

}
```
Thereby this implementation can be used as a Hessian service, it has to be defined as a Spring bean. Therefore we need a Spring configuration file `hello-world-service-config.xml`. The location for this file is `src/main/resources/META-INF/spring.` The bean configuration looks like the following code:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="helloWorldService" class="com.github.skosmalla.hello.world.spring.hessian.HelloWorldImpl"/>

</beans>
```

In this module we have two dependencies, one to the API module and one to the `spring-beans` module.

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
  </dependency>
  <dependency>
    <groupId>com.github.skosmalla.spring.hessian</groupId>
    <artifactId>hello-world-api</artifactId>
  </dependency>
</dependencies>
```

### The WAR Deployment

The module `hello-world-war`  describes the configuration for the server deployment. The artifact of this module becomes a WAR file. Therefore, the packaging of this Maven module is `war.`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.github.skosmalla.spring.hessian</groupId>
    <artifactId>hello-world-spring-hessian</artifactId>
    <version>1.0.0-SNAPSHOT</version>
  </parent>

  <artifactId>hello-world-war</artifactId>

  <name>Hello World WAR </name>

  <packaging>war</packaging>

  <build>
    <defaultGoal>install</defaultGoal>
  </build>
</project>
```

Now, we have to do two things for running our server application on a servlet container:

1.  Add configuration of Spring application context with our implementation to the servlet container.
2.  Add configuration to dispatch request to our Hessian service.

To create an `ApplicationContext` instance in a web application, we have to configure an `ContextLoaderListener` in our Web Application Deployment Descriptor (location: `src/main/webapp/WEB-INF/web.xml`):

```xml
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>classpath:META-INF/spring/*.xml</param-value>
</context-param>

<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

The `ContextLoaderListener` builds the root application context from all Spring configuration files located in the classpath under `META-INF/spring` (That pattern matches our `hello-world-service-config.xml`).  But no request can be processed. Therefore we have to configure a servlet that dispatch the request to the service. Here, the Spring Framework supports us with a `DispatcherServlet.` To use it we have to add that servlet to our Web Application Deployment Descriptor (location: `src/main/webapp/WEB-INF/web.xml`):

```xml
<servlet>
  <servlet-name>hessian</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
  <servlet-name>hessian</servlet-name>
  <url-pattern>/hessian/*</url-pattern>
</servlet-mapping>
```
 That configuration means that servlet is named as `hessian` and it is responsible for all request to the URL `http://<URL to the Tomcat instanz>/<webapp-context>/hessian/*` . Now, we have to configure the Hessian service interface that are dispatched by the servlet `hessian`. For it, we have to add a Spring configuration file in the same location like Web Application Deployment Descriptor (`src/main/webapp/WEB-INF`). The name of that file have to be `hessian-servlet.xml` (the pattern is `<servlet name>-servlet.xml`). Here, we configure the Hessian service interface:

```xml
<bean name="/HelloWorldService" class="org.springframework.remoting.caucho.HessianServiceExporter">
  <property name="service" ref="helloWorldService" />
  <property name="serviceInterface" value="com.github.skosmalla.hello.world.spring.hessian.HelloWorld" />
</bean>
```

That Spring configuration file defines a new application context. It is child application context of the root application context, loaded by the `ContextLoaderListener.` A child application context can see every bean of the root application context, but the root application context cannot see beans of its child application context (for more information, have look at the [Spring Framework reference](http://static.springsource.org/spring/docs/3.1.1.RELEASE/spring-framework-reference/html/remoting.html#remoting-caucho-protocols-hessian)). The `HessianServiceExporter` has a reference to the service implementation, defined in the root application context. The URL of that Hessian service interface is `http://<URL to the Tomcat instanz>/<webapp-context>/hessian/HelloWorldService` (Pattern is `http://<URL to the Tomcat instanz>/<webapp-context>/hessian/<bean name of the HessianServiceExporter>`). So that these configurations can run in a servlet container, we have to add the dependencies, that contains `HessianServiceExporter,` `DispatcherServlet` and `ContextLoaderListener` to the `pom.xml`:

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
</dependency>
<dependency>
  <groupId>com.github.skosmalla.spring.hessian</groupId>
  <artifactId>hello-world-impl</artifactId>
</dependency>
<dependency>
  <groupId>com.caucho</groupId>
  <artifactId>hessian</artifactId>
</dependency>
```
The Hessian dependency is needed at runtime and the hello-world-impl contains our business logic and the spring configuration file for the root application context. With a `mvn clean install` Maven builds a WAR file in the project's `target` folder. This WAR file can be deployed on a Tomcat.

### The Hessian Test Client With Spring

Now we write a client to test the Hessian service. Therefore, we create a new Maven module.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.github.skosmalla.spring.hessian</groupId>
    <artifactId>hello-world-spring-hessian</artifactId>
    <version>1.0.0-SNAPSHOT</version>
  </parent>

  <artifactId>hello-world-client</artifactId>

  <name>Hello World Client</name>

</project>
```

Spring Framework offers a `HessianProxyFactoryBean` for calling the remote `HelloWorld` service. The configuration for this `HessianProxyFactoryBean` could look like the following code snippet:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

<bean id="helloWorldService"
  class="org.springframework.remoting.caucho.HessianProxyFactoryBean">
  <property name="serviceUrl"
    value="http:/localhost:8080/hello-world/hessian/HelloWorldService" />
  <property name="serviceInterface"
    value="com.github.skosmalla.hello.world.spring.hessian.HelloWorld" />
</bean>
</beans>
```
In the property `serviceInterface` we define the interface of the Hessian service, here `com.github.skosmalla.hello.world.spring.hessian.HelloWorld.` In the property `serviceUrl` we define the URL to the Hessian Service deployed on the Tomcat. In our sample the Tomcat is on `localhost` with port number `8080` and the web application is `hello-world`.

Now, this factory bean creates Hessian service proxy for us:

```java
package com.github.skosmalla.hello.world.spring.hessian;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class HessianClient {

  public static void main(String[] args) {
    ApplicationContext appContext = new ClassPathXmlApplicationContext("META-INF/spring/hessian-config.xml");

    HelloWorld service = (HelloWorld) appContext.getBean("helloWorldService");

    String welcomeMessage = service.welcome();

    System.out.println(welcomeMessage);

  }
}
```

The dependencies for the client are the following one:

```xml
<dependencies>
  <dependency>
    <groupId>com.github.skosmalla.spring.hessian</groupId>
    <artifactId>hello-world-api</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
  </dependency>
  <dependency>
    <groupId>com.caucho</groupId>
    <artifactId>hessian</artifactId>
    <scope>runtime</scope>
  </dependency>
</dependencies>
```
If we start this client we will get "Hello World" on our command line. Now, we have seen a full example how to set up a Hessian service in a Spring web application and how to call such a service remotely. The full code you can find on [Github](https://github.com/skosmalla/maven-spring-hessian).

### Links

*   [Spring Framework Reference - Hessian Service](https://github.com/skosmalla/maven-spring-hessian)
*   [The Full Hello World Project on GitHub](https://github.com/skosmalla/maven-spring-hessian)
*   [Spring Framework Reference - Spring Web Application](http://static.springsource.org/spring/docs/3.1.1.RELEASE/spring-framework-reference/html/beans.html#context-create)
*   [Maven War Plugin](http://maven.apache.org/plugins/maven-war-plugin/)
