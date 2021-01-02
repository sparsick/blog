---
title: 'Automated Build of RCP Artifacts with Maven Tycho - A field report (Part 2)'
date: 2012-02-24
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
  - osgi

# comment: false # Disable comment if false.
---

In my last post of this series, I described how to solve the dependency problem. In this post, I will describe how to build Eclipse Plugin and Eclipse Feature with Maven Tycho and which problems I have.

### POM Definition

The procedure for both Eclipse artifacts is similar. In both, you have to insert a `pom.xml` and fill it with the Maven coordinate (group id, artifact id and version) and the packaging type. But there exists some rules about the content.

The packaging type is depended on what Eclipse artifact should be built:

| Eclipse Artifact | Packaging Type |
| ---------------- | ---------------|
| Eclipse Plugin   | eclipse-plugin |
| Eclipse Feature  | eclipse-feature|

The version specification in the POM is depended on the version specification in the manifest (in case of Eclipse plugin) and by the version specification in the `feature.xml` (in case of Eclipse feature) respectively:

| MANIFEST.MF / feature.xml | pom.xml        | Description        |
| ------------------------- | -------------- | ------------------ |
| 1.5.1.qualifier           | 1.5.1-SNAPSHOT | Development version|
| 1.5.1                     | 1.5.1          | Release version    |

The specification of the artifact id in the POM must be equal like the bundle symbolic name in the manifest (in case of Eclipse plugin) and by the specification of the id in the `feature.xml` (in case of Eclipse feature) respectively. That's all you need to define the `pom.xml`. Then call only `mvn clean install` and the Eclipse artifact is built by Maven Tycho.

### Troubleshooting

I met two problems during the introduction of Maven Tycho for the building of Eclipse plugins. The first problem was that Maven Tycho sometimes throws a `NullPointerException` during the read-out of the manifest file. The reason is that a blank must be between the colon, that follows after the manifest header, and the command.

```
# Don't
Import-Package:com.library.*
# Do
Import-Package: com.library.*
```

The second problem was that Maven Tycho throws compiler error although everything is alright in Eclipse. The analyze of the error message shows that the command for the `Import-Package` in the manifest is not completed. A second analyze shows that package names of used Eclipse RCP artifacts are missing. After adding these missing package names in the manifest, Maven Tycho builds without error. But this also means that Eclipse does not generate the manifest osgi-compliantly.
