---
title: 'How to Configure Apache2 as Forward and Reverse Proxy'
date: 2017-08-25
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
  - Linux
tags:
  - apache2
  - debian
  - forwarding
  - linux
  - proxy
  - rewarding
  - ubuntu
  - virtual-host
# comment: false # Disable comment if false.
---
This is a cook recipe to configure an Apache2 as a forward and reverse proxy on Debian-based Linux systems like Ubuntu or Debian itself.

Installation
============

It is assumed that the `apache2` package is already installed on your system. For the proxy feature, we have to install the Apache2 module `libapache2-mod-proxy-html` on the system and activate theses Apache modules. At the end, Apache2 has to be restarted, so that the modules can be used.

```shell
sudo apt-get install libapache2-mod-proxy-html
sudo a2enmod proxy
sudo a2enmod proxy_html
sudo a2enmod proxy_http
sudo service apache2 restart
```


Configuration Forwarding and Rewarding
======================================

We want to forward the URL request `http://jenkins.mycompany.com` to `http://jenkins.mycompany.com:8083` and rewarding `http://jenkins.mycompany.com:8083` to `http://jenkins.mycompany.com` for every response. For that, we have to create a so-called `Virtual Host` in Apache2. It is easiest to copy the configuration of the default one and adjust it.

```shell
cd /etc/apache2/sites-available
sudo cp 000-default.conf jenkins_ci.conf
```

It is best practice to create one conf file per Virtual Host. Adding and removing Virtual Host is easier with this approach.

The content of the Virtual Host configuration should look similar to the following one:


```shell
<VirtualHost jenkins.mycompany.com:80>
  ServerName jenkins.mycompany.com

  ServerAdmin webmaster@localhost

  ProxyRequests Off
  <Proxy *>
    Order deny,allow
    Allow from all
  </Proxy>
  ProxyPass / http://localhost:8083/
  ProxyPassReverse / http://localhost:8083/

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
We have to add the host name (in this case `jenkins.my.company.com`) to the line 1 and 2, so that Apache2 knows for which host name is this Virtual Host. In line 11 and 12 the mapping is configured (in this case, everything that call the host name directly should be forwarded to `http://localhost:8083;` the opposite for rewarding). At the end this configuration has to be enabled.

```shell
sudo a2ensite jenksin_ci.conf
sudo service apache2 reload
```
 

Further Information
===================

1.  https://wiki.ubuntuusers.de/Apache/mod_proxy_html/
2.  https://wiki.ubuntuusers.de/Apache/Virtual_Hosts/
