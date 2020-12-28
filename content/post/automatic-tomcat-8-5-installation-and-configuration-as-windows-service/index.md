---
title: 'Automatic Tomcat 8.5 Installation and Configuration as Windows Service'
date: 2017-03-15
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
  - Continuous Integration
  - JVM
  - Windows
tags:
  - automation
  - cmd
  - configuration
  - Continuous Integration
  - installation
  - JVM
  - tomcat
  - Windows

# comment: false # Disable comment if false.
---

If you want to install Tomcat on Windows system as a service, you'll get the recommendation to use the _32-/64-Bit Windows Service Installer_. If you want to install the Tomcat manually, it's fine. But you can't use this installer for an automatic installation and configuration of Tomcat, because the installer is UI-based. The next sections explain how you can install and configure Tomcat on a CMD.

Tomcat Installation
===================

1.  Download from the Apache Tomcat 8.5 [download page](http://tomcat.apache.org/download-80.cgi) the `Core 64-bit Windows zip` (or the 32-Bit zip).
2.  Unzip it (for example to `C:\tomcat\`)

That's it. Now we have a ready-to-use Tomcat with default configuration values. But it isn't install as a service.

Installation and Configuration As Windows Service
=================================================

1.  Go to the `bin` folder in the installation folder of Tomcat (in the example  it's `C:\tomcat\apache-tomcat-8.5.11\bin`)
2.  Install Tomcat as service named `tomcat8` by calling `service.bat install <servicename>`
```powershell
C:\tomcat\apache-tomcat-8.5.11\bin>service.bat install tomcat8
```
3.   `tomcat8.exe //US//<servicename>` followed by configuration parameter configures the Tomcat service. For example:
```powershell
C:\tomcat\apache-tomcat-8.5.11\bin>tomcat8.exe //US//tomcat8 --Startup=auto --JavaHome="C:\Program Files\Java\jre1.8.0_112" --JvmMs=2048 --JvmMx=4096 ++JvmOptions=-Dkey=value
```
4.  Start the Tomcat service with `net start <servicename>`
```powershell
net start tomcat8
```
5.  You can check on http://localhost:8080 whether Tomcat is installed correctly.

The configuration example (step 3) shows how to configure the JVM (heap space, Java option etc.), where Java is installed and which start type should use for the service. The full list of the possible configuration parameter for the Tomcat service can be found in the Apache Tomcat's [Windows Service documentation](http://tomcat.apache.org/tomcat-8.5-doc/windows-service-howto.html#Command_line_parameters). Now we have everything together for writing a Powershell script that does these steps automatically.
