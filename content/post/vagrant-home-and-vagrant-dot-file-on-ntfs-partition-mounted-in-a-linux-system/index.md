---
title: 'Vagrant Home and Vagrant Dot File On NTFS Partition Mounted In A Linux System'
date: 2014-11-18
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
  - Vagrant
tags:
  - linux
  - mount
  - ntfs
  - vagrant

# comment: false # Disable comment if false.
---

I use Vagrant together with VirtualBox  on an Ubuntu based Linux system.  Because my internal SSD drive isn't so large, I outsource the location of VirtualBox's VMs to an external HDD drive with a NTFS partition. Additionally, I set Vagrant's environment variables `VAGRANT_DOTFILE_PATH` and `VAGRANT_HOME` so, that the directories `.vagrant` and `.vagrant.d` are also on the external HDD drive. External HDD drive with NTFS partition are auto-mounted with the following mount options on an Ubuntu based Linux system.

```bash
rw,auto,user,fmask=0111,dmask=0000
```

For typical Vagrant commands like `vagrant up, vagrant destroy, vagrant halt` these mount options are sufficient. After using Vagrant for a while, I wanted to install some Vagrant plugins. The command for plugin installation in Vagrant is `vagrant plugin install <plugin name>.` The plugin installation failed with following error message.

```bash
An error occurred while installing nokogiri (1.6.3.1), and Bundler cannot continue.
Make sure that gem install nokogiri -v '1.6.3.1' succeeds before bundling.
ERROR vagrant: Bundler, the underlying system Vagrant uses to install plugins,
reported an error. The error is shown below. These errors are usually
caused by misconfigured plugin installations or transient network
issues.
```

A hint on the [Vagrant project's issue tracker](https://github.com/mitchellh/vagrant/issues/4582) brings me to try out several mount options for the NTFS partition. Finally, the NTFS partition has to be mounted with following mount option and then the plugin installation is successful.

```bash
rw,auto,user,uid=skosmalla,guid=skosmalla,exec,fmask=0022,dmask=0000
```

### Links

*   [Ubuntu documentation about mounting partitions automatically](https://help.ubuntu.com/community/AutomaticallyMountPartitions)
