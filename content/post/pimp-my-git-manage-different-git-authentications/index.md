---
title: 'Pimp My Git - Manage Different Git Authentications'
date: 2022-02-21
#description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
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
  - Git
tags:
  - git
  - ssh
  - authentication

# comment: false # Disable comment if false.
---
Sometimes you have to work on different projects that are hosted on different Git management systems (GitHub, Gitlab, BitBucket, Gitea).
The classical scenario is that you work on open source projects that are hosted on GitHub and on enterprise projects that are hosted on an own Git management system.
These projects also need different authentications.
These authentications can be managed in Password Manager.
But it remains cumbersome to log in with username and password.
Especially with Git repositories, there is the possibility to authenticate using an SSH key and SSH can be configured to detect itself which key to use.

Let's say we have two Git projects that need two different authentication. Both projects are accessible under two different domains, **example1.com** and **example2.com**. For each of the two projects, we generate a public/private SSH key pair (`id_example1` and `id_example2`).

```shell
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/sparsick/.ssh/id_rsa): /home/sparsick/.ssh/id_example1
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/sparsick/.ssh/id_example1.
Your public key has been saved in /home/sparsick/.ssh/id_example1.pub.
The key fingerprint is:
SHA256:9XVMIDlWX/6OvAzVmqKUBm8pmNNoiFkjJZ/+RERhp/A sparsick@sparsick-ThinkPad-T14
The key's randomart image is:
+---[RSA 2048]----+
| . +.. .oo.o|
| = o +. =.|
| . . E .. .. =|
| + o . . .
| . = . S . . o|
| * + = o o = |
| o o B o = .|
| + . = . + . |
| . . o |
+----[SHA256]-----+
```

The private key is then located in the file `id_example1` and the public key in the file `id_example1.pub`. This public key must be stored in the respective Git management system or Git server, so that the authentication via SSH can work.

Now it must be entered in the SSH configuration for which domain which key is to be used. For this, if not existing, a file config is created under `$USER_HOME/.ssh` and in it is entered for which domain which key is valid:

```shell
Host example1.com
HostName example1.com
User git
IdentityFile ~/.ssh/id_example1

Host example2.com
HostName example2.com
User git
IdentityFile ~/.ssh/id_example2
```

Thus, the git repositories can be cloned and pushed without entering the respective credentials.
