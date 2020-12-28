---
title: 'Configuration over JNDI in Spring Framework'
date: 2014-10-04
#description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Java
  - Spring Framework
tags:
  - Java
  - jndi
  - spring
  - Spring Framework

# comment: false # Disable comment if false.
---

From a certain point on, an application has to be configurable.  Spring Framework has a nice auxiliary tool for this issue since the first version 0.9 , the class [`PropertyPlaceholderConfigurer`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/config/PropertyPlaceholderConfigurer.html "JavaDoc PropertyPlaceholderConfigurer") and since Spring Framework 3.1 the class [`PropertySourcesPlaceholderConfigurer.`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/support/PropertySourcesPlaceholderConfigurer.html "JavaDoc PropertySourcesPlaceholderConfigurer") When you start a Google search for `PropertyPlaceholderConfigurer,` you will find many examples where the configuration items are saved in properties files. But in many Java enterprise applications, it is common that the configuration items are loaded over [JNDI](http://www.oracle.com/technetwork/java/index-jsp-137536.html "Oracle's Official Homepage about JNDI") look ups. I'd like to demonstrate how the `PropertyPlaceholderConfigurer` (before Spring Framework 3.1) and accordingly `PropertySourcesPlaceholderConfigurer` (since Spring Framework 3.1) can help to ease the configuration over JNDI look ups in our application.

Initial Situation
-----------------

We have an web application that has a connection to a database. This database connection has to be configurable. The configuration items are defined in a web application context file.

context.xml
```xml
<Context docBase="/opt/tomcat/warfiles/jndi-sample-war.war" antiResourceLocking="true">
  <Environment name="username" value="demo" type="java.lang.String" override="false"/>
  <Environment name="password" value="demo" type="java.lang.String" override="false"/>
  <Environment name="url" value="jdbc:mysql://192.168.56.101:3306/wicket_demo" type="java.lang.String" override="false"/>
</Context>
```

For loading these configuration items, the JNDI look up mechanism is used. In our application we define a data source bean in a  Spring context XML file. This bean represents the database connection.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
     http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context.xsd">

  <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close">
        <property name="url" value="${url}" />
        <property name="username" value="${username}" />
        <property name="password" value="${password}" />
  <bean>
</beans>
```
Every value that starts and ends with `${}` should be replaced by `PropertyPlaceholderConfigurer` and accordingly `PropertySourcesPlaceholderConfigurer` at the time when launching the application. The next step is to set up `PropertyPlaceholderConfigurer` and accordingly `PropertySourcesPlaceholderConfigurer.`

Before Spring Framework 3.1 - `PropertyPlaceholderConfigurer` Set Up for JNDI Look Up
-------------------------------------------------------------------------------------

We define a `PropertyPlaceholderConfigurer ` bean  in a Spring context XML file. This bean contains to an inner bean that maps the property names of the data source bean to the corresponding JNDI name. The JNDI name consists of two parts. The first part is the name of the context in which the resource is (in our case `java:comp/env/`) and the second part is the name of the resource (in our case either `username, password` or `url).`

```xml
<bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="properties">
        <bean class="java.util.Properties">
            <constructor-arg>
                <map>
                    <entry key="username">
                        <jee:jndi-lookup jndi-name="java:comp/env/username" />
                    </entry>
                    <entry key="password">
                        <jee:jndi-lookup jndi-name="java:comp/env/password" />
                    </entry>
                    <entry key="url">
                        <jee:jndi-lookup jndi-name="java:comp/env/url" />
                    </entry>
                </map>
            </constructor-arg>
        </bean>
    </property>
</bean>
```

Since Spring Framework 3.1 - `PropertySourcesPlaceholderConfigurer` Set Up for JNDI Look Up
-------------------------------------------------------------------------------------------

Since Spring 3.1 `PropertySourcesPlaceholderConfigurer` should be used instead of `PropertyPlaceholderConfigurer.` This effects that since Spring 3.1 the <context:property-placeholder/> namespace element registers an instance of `PropertySourcesPlaceholderConfigurer` (the namespace definition must be spring-context-3.1.xsd) instead of `PropertyPlaceholderConfigurer` (you can simulate the old behaviour when you use the namespace definition spring-context-3.0.xsd). So our Spring XML context configuration is very short, when you comply some convention (based on the principle `Convention over Configuration).`

```xml
<context:property-placeholder/>
```
The default behavior is that the `PropertySourcesPlaceholderConfigurer` iterates through a set of `[PropertySource](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/core/env/PropertySource.html "JavaDoc PropertySource")` to collect all properties values. This set contains [`JndiPropertySource`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jndi/JndiPropertySource.html "Javadoc JndiPropertySource") per default in a Spring based web application. By default, `JndiPropertySource` looks up after JNDI resource names prefixed with `java:comp/env`. This means if your property is `${url}`, the corresponding JNDI resource name has to be `java:comp/env/url`. The source code of the sample  web application is hosted on [GitHub](https://github.com/skosmalla/application-configuration-spring-maven "Sample Source Code").
