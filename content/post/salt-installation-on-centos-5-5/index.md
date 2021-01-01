---
title: 'Salt Installation On Centos 5.5'
date: 2014-10-18
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
  - Configuration Management
  - Salt
tags:
  - centos5
  - python
  - salt
  - saltstack
  - zeromq

# comment: false # Disable comment if false.
---

When you follow the instruction step in the [installation guide](http://docs.saltstack.com/en/latest/topics/installation/rhel.html#installation-from-epel "Salt Installation Guide for Centos") for Centos 5.5, the package manager automatically installs ZeroMQ in version 2.2. This ZeroMQ version [makes some troubles](http://docs.saltstack.com/en/latest/topics/troubleshooting/#salt-master-stops-responding "Salt Troubleshooting for ZeroMQ"). Therefore, Salt recommends using ZeroMQ in version >= 3.2. There is no ZeroMQ package for this version available in the official Centos 5 repositories, neither in EPEL. So the community prepares some RPMs for Centos 5 for installing ZeroMQ in version 3.3.2.2. This post describes which steps has to be done to install Salt 2014.1.11 on a Centos 5.5 64bit.

Common Installation Steps on Salt Master Node and Salt Minion Nodes
-------------------------------------------------------------------

1.  EnableEPEL repository `sudo yum install epel-release`
2.  Install the Python 2.6 development package `sudo yum install python26-devel.x86_64`
3.  Installl all RPMs listed on [this site.](http://docs.saltstack.com/downloads/cent5/ "Alternative RPMs for ZeroMQ on Centos 5")
```bash
sudo rpm -Uvh http://docs.saltstack.com/downloads/cent5/libzmq3-3.2.2-13.1.x86_64.rpm
sudo rpm -Uvh http://docs.saltstack.com/downloads/cent5/python-zmq-debuginfo-13.1.0-1.x86_64.rpm
sudo rpm -Uvh http://docs.saltstack.com/downloads/cent5/python26-zmq-13.1.0-1.x86_64.rpm
sudo rpm -Uvh http://docs.saltstack.com/downloads/cent5/python26-zmq-tests-13.1.0-1.x86_64.rpm
sudo rpm -Uvh http://docs.saltstack.com/downloads/cent5/zeromq-3.2.2-13.1.x86_64.rpm
sudo rpm -Uvh http://docs.saltstack.com/downloads/cent5/zeromq-debuginfo-3.2.2-13.1.x86_64.rpm
sudo rpm -Uvh http://docs.saltstack.com/downloads/cent5/zeromq-devel-3.2.2-13.1.x86_64.rpm
```

Specific Installation Steps on Salt Master Node
-----------------------------------------------

Now, we can follow the official installation steps.

1.  Install Salt Master `sudo yum install salt-master`

Specific Installation Steps on Salt Minion Node
-----------------------------------------------

Again, we can follow the official installation step.

1.  Install Salt Minion `sudo yum install salt-minion`

Configuration
-------------

This section describes only the important configuration issues for running the first command from a master to its  minions. For the whole configuration possibilities, please check the [Salt configuration documentation](http://docs.saltstack.com/en/latest/ref/configuration/index.html "Salt Configuration Documentation"). For a successful communication between master and minions, two configuration are important.

*   Set up the firewall on the master side and
*   key exchange between master and minions (because the communication is encrypted).

### Firewall Configuration

By default Salt listens on ports 4505 and 4506. Therefore, the firewall has to be configured to accept incoming communication on these ports.

1.  Open `/etc/sysconfig/iptables` as root with your favourite editor.
2.  Add following lines `-A INPUT -m state --state new -m tcp -p tcp --dport 4505 -j ACCEPT -A INPUT -m state --state new -m tcp -p tcp --dport 4506 -j ACCEPT`
3.  Restart the service iptables `sudo service iptables restart`

### Key Exchange Configuration

The master can only send commands to minion whose keys are accepted by the master.

1.  Start the minion on the minion node. ` salt-minion `
2.  Ensure that the master runs on the master node. ` salt-master `
3.  On the master node, look which keys aren't accepted ` salt-key -L `
4.  To accept all unaccepted key call on the master node ` salt-key -A `
5.  To test whether the minion is available by the master, call on the master node ` salt name-of-minion test.ping `

Further Information
-------------------

*   [Salt Installation Guide](http://docs.saltstack.com/en/latest/topics/installation/index.html)
*   [Salt Troubleshooting](http://docs.saltstack.com/en/latest/topics/troubleshooting/index.html#salt-master-stops-responding)
*   [RPM list for ZeroMQ 3.3 on Centos 5](http://docs.saltstack.com/downloads/cent5/)
*   [Salt Configuration Documentation](http://docs.saltstack.com/en/latest/ref/configuration/index.html)
*   [Salt Documentation](http://docs.saltstack.com/en/latest/contents.html)
