---
title: 'Test Environment for Ansible on a Windows System Without Linux Subsystem Support'
date: 2020-01-24
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
  - Ansible
  - Configuration Management
  - Continuous Integration
  - Linux
  - Quality Assurance
  - Testing
  - Vagrant
  - Windows
tags:
  - ansible
  - test-infrastructure
  - vagrant
  - virtualbox
  - windows


# comment: false # Disable comment if false.
---

Some weeks ago, I gave a workshop about Ansible and I was asked how to set up a local test environment for Ansible. Requirement is that this test environment can run on a Windows 10 without a Linux subsystem. I don't why, but it was not my first customer where Windows 10 could not be used with Linux subsystem (Don't ask me about reasons). So I recommend following set up.

Idea
----

Ansible itself requires a Linux-based system as the control machine (so-called `control node`). The machine that is provisioned (so-called `managed node`) can be both, Linux- or Windows-based. In my following example, the manage node is Linux-based.   So when I have to develop on a Windows machine, I install two Linux-based virtual machines. One for calling the Ansible's playbooks and a second one for testing the provisioning. I set up the virtual machines with VirtualBox and Vagrant. Vagrant allows me to share the playbooks easily between host and virtual machines. That means, I can develop the playbook on the Windows system and the virtual machines can run headless. The next section shows you how to set up this tool chain.

Tool Chain Setup
----------------

At first, install [VirtualBox](https://www.virtualbox.org/) and [Vagrant](https://www.vagrantup.com/) on your machine. I additionally use [Babun](http://babun.github.io/), a windows shell based on Cygwin and oh-my-zsh, for a better shell experience on Windows, but this isn't necessary. Then, go to the directory (let's called it `ansible-workspace`), where your Ansible's playbooks are located. Create there a Vagrant configuration file with the command `vagrant init`:

```shell
$ cd ansible-workspace
$ vagrant init
$ tree  
.
├── deploy-app.yml
├── host_vars
│   └── ubuntu-server
├── inventories
│   ├── production
│   └── test
├── roles
│   ├── deploy-on-tomcat
│   │   ├── defaults
│   │   │   └── main.yml
│   │   └── tasks
│   │       ├── cleanup-webapp.yml
│   │       ├── deploy-webapp.yml
│   │       ├── main.yml
│   │       ├── start-tomcat.yml
│   │       └── stop-tomcat.yml
└── Vagrantfile
```

Now, we have to choose a so-called Vagrant Box on [Vagrant Cloud](https://app.vagrantup.com/boxes/search). A box is the package format for a Vagrant environment. It depends on the provider and the operating system that you choose to use. In our case, it is a VirtualBox VM image based on a minimal Ubuntu 18.04 system (box name is [bento/ubuntu-18.04](https://app.vagrantup.com/bento/boxes/ubuntu-18.04) ). In our case, we have to configure two boxes in our Vagrantfile. Both are based on bento/ubuntu-18.04.

```ruby
Vagrant.configure("2") do |config|  
  config.vm.define "managed-node" do |ubuntu|
    ubuntu.vm.box = "bento/ubuntu-18.04"
  end

  config.vm.define "control-node" do |vbox|
     vbox.vm.box = "bento/ubuntu-18.04"
  end
end
```

The next step is to ensure that Python is installed on the managed node and this node is available via an IP address. For that, we use the shell provisioning, and we configure a private network in the Vagrantfile:

```ruby
# ...  
  config.vm.define "managed-node" do |ubuntu|
    ubuntu.vm.box = "bento/ubuntu-18.04"
    ubuntu.vm.network "private_network", ip: "192.168.33.11"

    ubuntu.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install python -y
    SHELL
  end
# ...
```

In the control node, we have to ensure that Ansible is installed and that a SSH connection from control node to our managed node via SSH-key is possible. For that agian, we use the shell provisioning in the Vagrantfile:

```ruby
 # ...
  config.vm.define "controll-node" do |vbox|
     vbox.vm.box = "bento/ubuntu-18.04"

     vbox.vm.provision "shell", inline: <<-SHELL
       sudo apt-get update -y
       sudo apt-get install -y software-properties-common
       sudo apt-add-repository ppa:ansible/ansible
       sudo apt-get update -y
       sudo apt-get install -y ansible
     SHELL

     vbox.vm.provision "shell", privileged: false, inline: <<-SHELL
       ssh-keygen -t rsa -q -f "/home/vagrant/.ssh/id_rsa" -N ""
       ssh-keygen -R 192.168.33.11
       ssh-keyscan -t rsa -H 192.168.33.11 >> /home/vagrant/.ssh/known_hosts
       sshpass -p 'vagrant' ssh-copy-id -i /home/vagrant/.ssh/id_rsa.pub vagrant@192.168.33.11
     SHELL
  end
# ...
```

On Github's Gist you can find the whole [Vagrantfile](https://gist.github.com/sparsick/ee5aa272b046e0ea77dd41ef08134893).

The last step is to configure your Ansible project, so that the playbooks can run against the managed node. Therefore, you create a new inventory file called `local-test` with managed node's IP address:

```shell
$ cat inventories/local-test
[application_server] # name of your group that is mention in your playbook
192.168.33.11
```

Workflow
--------

After setting up the tool chain let's have a look how to work with it. I write my Ansible playbook on the Windows system and run them from the control node against managed node. First at all, we have to start our Vagrant boxes.

```shell
$ cd ansible-workspace
$ vagrant up
```

When both Vagrant boxes are ready to use, we can jump into control node box with:

```shell
$ vagrant ssh control-node
```

You can find the Ansible playbooks inside the box in the folder `/vagrant` .  In this folder run Ansible:

```
$ cd /vagrant
$ ansible-playbook -i inventories/local-test -u vagrant deploy-app.yml
```

It is important that you use the `local-test` inventory.

Outlook
-------

Some Docker fans would prefer a container instead of a virtual machine. But remember, Docker runs on Windows in a virtual machine, therefore, I don't see a benefit for using Docker instead of a virtual machine. But of course with Windows 10 native container support a setup with Docker is a good alternative if Ansible doesn't run on the Linux subsystem.

Do you another idea or approach? Let me know and write a comment.

Links
-----

1.  [VirtualBox](https://www.virtualbox.org/)
2.  [Vagrant](https://www.vagrantup.com/)
3.  Whole Vagrantfile on [GitHub](https://gist.github.com/sparsick/b36b81c291c556ebdcc94f224136abc8).
