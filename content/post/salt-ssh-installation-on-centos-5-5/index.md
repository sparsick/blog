---
title: 'Salt SSH Installation on Centos 5.5'
date: 2014-11-01
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
  - agentless
  - centos5
  - linux
  - python
  - salt
  - saltstack
  - ssh


# comment: false # Disable comment if false.
---

Salt has the option to manage servers agentlessly.  Agentless means that the targets don't need a agent process. The master orchestrates the target system over SSH. Therefor it exists an own command called _salt-shh. _ The following sections explain how to install Salt SSH on a CentOs 5.5 and how to configure minimally a master and its targets for a test connection. The how to is tested with Salt version 2014.1.11.

Installation
------------

### On Master Node

Salt SSH is a part of the master package, so we have to install salt-master.
```bash
sudo yum install salt-master
```
It also installs the optional dependencies. For an agent mode these dependencies can make trouble (see for more information [Salt Installation on Centos 5.5](https://blog.sandra-parsick.de/2014/10/18/salt-installation-on-centos-5-5/ "Salt Installation On Centos 5.5")). But for our case these dependencies can be ignored because the communication between master and target systems is over SSH.

### On Target Nodes

On target nodes, we have to ensure that Python 2.6. are installed and some Python 2.6 modules (see [Salt Dependency page](http://docs.saltstack.com/en/latest/topics/installation/#dependencies "Salt Dependencies")). These are needed because the master copies Python scripts to the minions and run them on the targets. So the following steps has to be done.

1.  Enable EPEL Release `sudo yum install epel-release `

2.  Install Python 2.6 package and needed Python modules `sudo yum install python26 python26-msgpack python26-PyYAML python26-jinja2 python26-markupsafe python-libcloud python26-requests `

Configuration
-------------

This section describes only the important configuration issues for running the first command from a master to its targets. For further configuration possibilities, please read the [Salt documentation](http://docs.saltstack.com/en/latest/topics/ssh/index.html) about configuration. The configuration depends whether the authentication uses password or public/private keys.

### Password Authentication

1.  Go on target nodes.
2.  Enable SSH password authentication.
    1.  Open `etc/ssh/sshd_config` with your favorite editor.
    2.  Ensure that the line `PasswordAuthentication yes` is active.
    3.  Restart SSH. `sudo service ssh restart `
3.  Go on master node.
4.  Configure the connection to the targets.
    1.  Open `/etc/salt/roster` with your favorite editor.
    2.  Add for every target following content
    ```text
    <Salt ID>:   # The id to reference the target system with
    host:    # The IP address or DNS name of the remote host
    user:    # The user to log in as
    passwd:  # The password to log in with
    ```
3.  Save the file.
4.  Test the communication. `salt-ssh <Salt ID> test.ping `

### Public/Private Key Authentication

1.  Go on the master node.
2.  Prepare SSH for key authentication
    1.  Call `ssh-keygen `
    2.  Reply following question
    ```bash
     Enter file in which to save the key (/home/skosmalla/.ssh/id_rsa):
     Enter passphrase (empty for no passphrase):
     Enter same passphrase again:
    ```
    3.  Keep the following information in mind.
    ```bash
    Your identification has been saved in /home/skosmalla/.ssh/id_rsa.
    Your public key has been saved in /home/skosmalla/.ssh/id_rsa.pub.
    The key fingerprint is:
    44:3e:ef:58:94:15:52:c2:88:ca:ab:21:43:53:3d:42 skosmalla@computer
    ```
    4.  Copy the public key (in our example id\_rsa.pub) to the targets. `ssh-copy-id -i /home/skosmalla/.ssh/id_rsa.pub username@target_host `
    5.  Check, if the ssh access is working without password. `ssh username@target_host `
3.  Configure the connection to the targets.
    1.  Open `/etc/salt/roster` with your favorite editor.
    2.  Add for every target following content.
    ```text
    <Salt ID>:   # The id to reference the target system with
    host:    # The IP address or DNS name of the remote host
    user:    # The user to log in as
    priv:    # File path to ssh private key, defaults to salt-ssh.rsa, in our example it is /home/skosmalla/.ssh/id_rsa.
    ```
4.  Test the communication. `salt-ssh <Salt ID> test.ping `

 Further Information
--------------------

*   [Salt SSH documentation](http://salt.readthedocs.org/en/latest/topics/ssh/index.html)
*   [Salt Rosters documentation](http://salt.readthedocs.org/en/latest/topics/ssh/roster.html)
