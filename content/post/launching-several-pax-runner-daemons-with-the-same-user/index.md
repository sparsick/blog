---
title: 'Launching Several Pax Runner Daemons With The Same User'
date: 2012-08-16
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
  - OSGi
tags:
  - daemon
  - deployment
  - osgi
  - pax
  - pax-runner

# comment: false # Disable comment if false.
---

I use [Pax Runner](http://team.ops4j.org/wiki/display/paxrunner/Pax+Runner "Homepage PAX Runner") to provisioning my OSGi applications. I want to use several OSGi applications on my test server. As soon as you start the second OSGi application provisioned by Pax Runner, you get the following exception:

```bash
Exception in thread "main" java.lang.RuntimeException: org.ops4j.pax.runner.daemon.lock exists. Please make sure that the Pax Runner daemon is not already running.
```
As per Google search it is a [bug](http://team.ops4j.org/browse/PAXRUNNER-416) in Pax Runner. The source code of  Pax Runner is hosted by [Github](https://github.com/ops4j/org.ops4j.pax.runner.git), so I create a [fork](https://github.com/rhenus-fl/org.ops4j.pax.runner) for fixing this issue. My first idea was to add a new runner option. Problem is that the method, which build the path for the daemon home directory, is static, so I have no chance to read the runner option without a big refactoring. I decided to add a new Java property `relative.runner.home`., so I need to change only one method `getRunnerHomeDir.` Now, the path of the daemon runner home directory is `user.home/.pax/relative.runner.home`. If this property is not set, the default path will be  `user.home/.pax/runner` (like the behaviour before this change). With this change, the start call of the Pax Runner looks like that:

```bash
java -Drelative.runner.home=app1 -cp pax-runner-1.7.6-SKM-PATCH.jar org.ops4j.pax.runner.daemon.DaemonLauncher --start
```
and the stop call of the Pax Runner looks like that:

```bash
java -Drelative.runner.home=app1 -cp pax-runner-1.7.6-SKM-PATCH.jar org.ops4j.pax.runner.daemon.DaemonLauncher --stop
```

For the second OSGi application, you have to change the value of `relative.runner.home`. But the second application will not start because of the following exception:

```bash
Exception in thread "main" java.lang.RuntimeException: Unable to set up shutdown port [8008].
at org.ops4j.pax.runner.daemon.Daemon.await(Daemon.java:253)
at org.ops4j.pax.runner.daemon.Daemon.start(Daemon.java:134)
at org.ops4j.pax.runner.daemon.Daemon.main(Daemon.java:84)
at org.ops4j.pax.runner.daemon.DaemonLauncher.start(DaemonLauncher.java:91)
at org.ops4j.pax.runner.daemon.DaemonLauncher.main(DaemonLauncher.java:69)
Caused by: java.net.BindException: Address already in use: JVM_Bind
at java.net.TwoStacksPlainSocketImpl.socketBind(Native Method)
at java.net.AbstractPlainSocketImpl.bind(AbstractPlainSocketImpl.java:376)
at java.net.TwoStacksPlainSocketImpl.bind(TwoStacksPlainSocketImpl.java:101)
at java.net.PlainSocketImpl.bind(PlainSocketImpl.java:175)
at java.net.ServerSocket.bind(ServerSocket.java:376)
at java.net.ServerSocket.&lt;init&gt;(ServerSocket.java:237)
at java.net.ServerSocket.&lt;init&gt;(ServerSocket.java:128)
at org.ops4j.pax.runner.daemon.Daemon.await(Daemon.java:250)
... 4 more
- Pax Runner daemon stopped.
```

Pax Runner uses a shutdown port to close the application, so every application needs its own shutdown port. You have two possibilities to set the shutdown port.

1.  You create a file called `runner.args` the same directory where the pax runner jar is and add the option `--org.ops4j.pax.runner.daemon.shutdown.port=8008` in this file.
2.  You get this option to the DaemonLauncher, directly:

```bash
 java -Drelative.runner.home=app2 -cp pax-runner-1.7.6-SKM-PATCH.jar org.ops4j.pax.runner.daemon.DaemonLauncher --start --org.ops4j.pax.runner.daemon.shutdown.port=8008
```

### Links

1.  [Pax Runner Homepage](http://team.ops4j.org/wiki/display/paxrunner/Pax+Runner)
2.  [Jira Bug Ticket](http://team.ops4j.org/browse/PAXRUNNER-416)
3.  [Pax Runner at Github](https://github.com/ops4j/org.ops4j.pax.runner.git)
4.  [Pax Runner Fork](https://github.com/rhenus-fl/org.ops4j.pax.runner)
