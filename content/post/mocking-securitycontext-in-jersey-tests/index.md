---
title: 'Mocking SecurityContext in Jersey Tests'
date: 2018-03-29
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
  - JVM
  - Quality Assurance
  - Testing
tags:
  - integration-testing
  - jersey
  - mock
# comment: false # Disable comment if false.
---


Jersey has a great possibility to write integration test for REST-APIs, written with Jersey. Just extend the class `JerseyTest` and go for it. I ran in an issue, where I had to mock a `SecurityContext`, so that the `SecurityContext` includes a special `UserPrincipal`. The challenge is that Jersey wraps the `SecurityContext` in an own class `SecurityContextInjectee` in tests. So I have to add my `SecurityContext` Mock to this Jersey's wrapper class. Let me demonstrate it in an example. Let say I have the following Jersey Resource:

```java
import javax.ws.rs.*;
import javax.ws.rs.core.*;

@Path("hello/world")
public class MyJerseyResource {

    @GET
    public Response helloWorld(@Context final SecurityContext context) {
        String name = context.getUserPrincipal().getName();
        return Response.ok("Hello " + name, MediaType.TEXT_PLAIN).build();
    }
}
```
In my test, I have to mock the `SecurityContext`, so that a predefined user principal can be used during the tests. I use Mockito as mocking framework. My mock looks like the following one

```java
import java.security.Principal;
import javax.ws.rs.core.SecurityContext;

import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;

final SecurityContext securityContextMock = mock(SecurityContext.class);
when(securityContextMock.getUserPrincipal()).thenReturn(new Principal() {
    @Override
    public String getName() {
        return "Alice";
    }
});
```
For adding this mocked `SecurityContext` to the wrapper class `SecurityContextInjectee`, I have to configure a `ResourceConfig` with a modified `ContainerRequestContext` in my Jersey Test. The mocked `SecurityContext` can be set in this modified `ContainerRequestContext` and then it will be used in the wrapper class:

```java
import java.security.Principal;

import javax.ws.rs.container.ContainerRequestContext;
import javax.ws.rs.container.ContainerRequestFilter;
import javax.ws.rs.core.SecurityContext;

import org.glassfish.jersey.server.ResourceConfig;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;

@Override
public Application configure() {
    final SecurityContext securityContextMock = mock(SecurityContext.class);
    when(securityContextMock.getUserPrincipal()).thenReturn(new Principal() {
        @Override
        public String getName() {
            return "Alice";
        }
    });

    ResourceConfig config = new ResourceConfig();
    config.register(new ContainerRequestFilter(){
        @Override
        public void filter(final ContainerRequestContext containerRequestContext) throws IOException {
            containerRequestContext.setSecurityContext(securityContextMock);
        }
    });
    return config;
}
```
Then, the whole test for my resource looks like the following one:

```java
import java.security.Principal;

import javax.ws.rs.container.ContainerRequestContext;
import javax.ws.rs.container.ContainerRequestFilter;
import javax.ws.rs.core.Application;
import javax.ws.rs.core.Response;
import javax.ws.rs.core.SecurityContext;

import org.apache.commons.httpclient.HttpStatus;
import org.glassfish.jersey.server.ResourceConfig;
import org.glassfish.jersey.test.JerseyTest;
import org.junit.Test;
import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;

public class MyJerseyResourceTest extends JerseyTest {

    @Test
    public void helloWorld() throws Exception {
        Response response = target("hello/world").request().get();

        assertThat(response.getStatus()).isEqualTo(HttpStatus.SC_OK);
        assertThat(response.getEntity()).isEqualTo("Hello Alice");
    }

    @Override
    public Application configure() {
        final SecurityContext securityContextMock = mock(SecurityContext.class);
        when(securityContextMock.getUserPrincipal()).thenReturn(new Principal() {
            @Override
            public String getName() {
                return "Alice";
            }
        });

        ResourceConfig config = new ResourceConfig();
        config.register(new ContainerRequestFilter() {
            @Override
            public void filter(final ContainerRequestContext containerRequestContext) throws IOException {
                containerRequestContext.setSecurityContext(securityContextMock);
            }
        });
        return config;
    }
}
```

Do you have a smarter solution for this problem? Let me know it and write a comment below.
