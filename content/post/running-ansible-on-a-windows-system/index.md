---
title: 'Running Ansible on a Windows System'
date: 2018-06-11
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
  - Vagrant
  - Windows
tags:
  - vagrant
  - virtualbox

# comment: false # Disable comment if false.
---

On my last conference talk (it was about [Ansible and Docker at DevOpsCon](https://www.sandra-parsick.de/talk/ansible-docker-devopscon/) in Berlin), I was asked what is the best way to run Ansible on a Windows system. Ansible itself requires a Linux-based system as the control machine. When I have to develop on a Windows machine, I install a Linux-based virtual machine to run the Ansible's playbooks inside the virtual machine. I set up the virtual machine with Virtualbox and Vagrant. This tools allow me to share the playbooks easily between host and the virtual machine. so I can develop the playbook on the windows system and the virtual machine can have a headless setup. The next section shows you how to set up this tool chain.

 Tool Chain Setup
-----------------

 At first, install [VirtualBox](https://www.virtualbox.org/) and [Vagrant](https://www.vagrantup.com/) on your machine. I additionally use [Babun](http://babun.github.io/), a windows shell based on Cygwin and oh-my-zsh, for a better shell experience on Windows, but this isn't necessary. Then, go to the directory (let's called it ansible-workspace), where your Ansible's playbooks are located. Create there a Vagrant configuration file with the command vagrant init:

```bash
ansible-workspace
├── inventories
│   ├── production
│   └── test
├── README.md
├── roles
│   ├── deploy-on-tomcat
│   │   ├── defaults
│   │   │   └── main.yml
│   │   └── tasks
│   │       ├── cleanup-webapp.yml
│   │       ├── deploy-webapp.yml
│   │       ├── main.yml
│   │       ├── start-tomcat.yml
│   │       └── stop-tomcat.yml
│   ├── jdk
│   │   └── tasks
│   │       └── main.yml
│   └── tomcat8
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       │   └── init.d
│       │       └── tomcat
│       ├── tasks
│       │   └── main.yml
│       └── templates
│           └── setenv.sh.j2
├── demo-app-ansible-deploy-1.0-SNAPSHOT.war
├── deploy-demo.yml
├── inventories
│   ├── production
│   └── test
├── roles
│   ├── deploy-on-tomcat
│   │   ├── defaults
│   │   │   └── main.yml
│   │   └── tasks
│   │       ├── cleanup-webapp.yml
│   │       ├── deploy-webapp.yml
│   │       ├── main.yml
│   │       ├── start-tomcat.yml
│   │       └── stop-tomcat.yml
│   ├── jdk
│   │   └── tasks
│   │       └── main.yml
│   └── tomcat8
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       │   └── init.d
│       │       └── tomcat
│       ├── tasks
│       │   └── main.yml
│       └── templates
│           └── setenv.sh.j2
├── setup-app-roles.yml
├── setup-app.yml
└── Vagrantfile

├── setup-app-roles.yml
├── setup-app.yml
└── Vagrantfile
```

Now, we have to choose a so-called Vagrant Box on [Vagrant Cloud](https://app.vagrantup.com/boxes/search). A box is the package format for a Vagrant environment. It depends on the provider and the operation system that you choose to use. In our case, it is a Virtualbox VM image based on a minimal Ubuntu 18.04 system (box name is [bento/ubuntu-18.04](https://app.vagrantup.com/bento/boxes/ubuntu-18.04) ). This box will be configured in our Vagrantfile:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-18.04"
end
```

The next step is to ensure that Ansible will be installed in the box. Thus, we use the shell provisioner of Vagrant. The Vagranfile will be extended by the provisioning code:

```ruby
Vagrant.configure("2") do |config|
  # ... other Vagrant configuration
  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update -y
    sudo apt-get install -y software-properties-common
    sudo apt-add-repository ppa:ansible/ansible
    sudo apt-get update -y
    sudo apt-get install -y ansible
    # ... other Vagrant provision steps
  SHELL
end
```

The last step is to copy the SSH credential into the Vagrant box. Thus, we mark the SSH credential folder of the host system as a Shared folder, so that we can copy them to the SSH config folder inside the box.

```ruby
Vagrant.configure("2") do |config|

  # ... other Vagrant configuration
  config.vm.synced_folder ".", "/vagrant"
  config.vm.synced_folder "path to your ssh config", "/home/vagrant/ssh-host"
  # ... other Vagrant configuration

  config.vm.provision "shell", inline: <<-SHELL
    # ... other Vagrant provision steps
    cp /home/vagrant/ssh-host/* /home/vagrant/.ssh/.
  SHELL
end
```

On Github's Gist you can found the whole [Vagrantfile](https://gist.github.com/sparsick/b36b81c291c556ebdcc94f224136abc8).

Workflow
--------

After setting up the tool chain let's have a look how to work with it. I write my Ansible playbook on the Windows system and run them from the Linux guest system against the remote hosts. For running the Ansible playbooks we have to start the Vagrant box.

```bash
cd ansible-workspace > vagrant up
```

When the Vagrant box is ready to use, we can jump into the box with:

```bash
vagrant ssh
```

You can find the Ansible playbooks inside the box in the folder `/vagrant`. In this folder run Ansible:

```bash
cd /vagrant
ansible-playbook -i inventories/test -u tekkie setup-db.yml
```

Outlook
-------

Maybe on Windows 10 it's possible to use Ansible natively, because of the Linux subsystem. But I don't try it out. Some Docker fans would prefer a container instead of a virtual machine. But remember, before Windows 10 Docker runs on Windows in a virtual machine, so therefore, I don't see a benefit for using Docker instead of a virtual machine. But of course with Windows 10 native container support a setup with Docker is a good alternative if Ansible doesn't run on the Linux subsystem.

Do you another idea or approach? Let me know and write a comment.

Links
-----

1.  [VirtualBox](https://www.virtualbox.org/)
2.  [Vagrant](https://www.vagrantup.com/)
3.  Whole Vagrantfile on [Github](https://gist.github.com/sparsick/b36b81c291c556ebdcc94f224136abc8).
