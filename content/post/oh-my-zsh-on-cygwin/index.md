---
title: "Using Cygwin in 2023"
date: 2023-11-30
description: "Installing oh-my-zsh on Cygwin" # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: true # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Windows
tags:
  - windows
  - cygwin
  - zsh
  - oh-my-zsh
  - linux
  - shell
  - how-to
# comment: false # Disable comment if false.
---

In one of my customer project I have to use a Windows 11 system. 
In 2023, it should be not a problem as a Linux fan to work on a Windows system, because I could install WSL2 and the story would end on this point. 
Because of reasons, it is not allowed and not possible to install WSL2 on my system, so I fall back to Cygwin. 

The default shell on Cygwin is a Bash, but I like to use ZSH in combination with the framework _oh-my-zsh_ and the theme _Starship_. 
Of course, Cygwin is not a 100% replacement for WSL2, but it simplifies many things, that I like to do in a shell.

In the next sections, I like to show you how to install oh-my-zsh on Cygwin, other goodies like installing a package manager for Cygwin, and which limits I found during using Cygwin.

Let's start with the installation of Cygwin.

## Install Cygwin
Go to the [homepage of Cygwin](https://cygwin.com/install.html) and download `setup-x86_64.exe`. If you don't have admin rights on your system, you have to use the powershell or CMD to execute the exe-file with a special flag:

```shell
setup-x86_64.exe --no-admin
```

On the package dialog, choose `zsh`, `git`, `unzip`, `curl` and `wget` for installing and finish the Cygwin installation.

## Set ZSH as default shell
As I said before, the default shell on Cygwin is Bash, so you have to set ZSH as the new default shell.

The easiest way is, to set your `SHELL` user environment variable. Search for __Edit Environment variables for your account__ to bring up the environment variables window, create a new variable named `SHELL` and give it the value `/usr/bin/zsh/`.

## Install oh-my-zsh
With the above preparations, installing oh-my-zsh on Cygwin is as easy as on a normal Linux environment. Call 

````shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
````

## Install Theme Starship

Normally, the installation works like in a normal Linux environment.
Run 

````shell
curl -sS https://starship.rs/install.sh | sh
````

and add the following line to the end of your `.zshr` file:

````shell
eval "$(starship init zsh)"
````

If the installation script failed, edit it and remove the if condition that let fails the installation. 

## Install Package Manager apt-cyg

If you don't want to call the setup.exe everytime, when you want to install a new package, you can install a package manager in Cygwin, called `apt-cyg`

````shell
wget rawgit.com/transcode-open/apt-cyg/master/apt-cyg
install apt-cyg /bin
````

After the installation you can install new package with 
````shell
apt-cyg install <package name>
````

## My found limits 
You can't use the current Azure cli on Cygwin, because a Python package for crypto needs Rust and Rust is [currently not supported](https://github.com/rust-lang/rust/issues/5526) in Cygwin out-of-the-box.
Therefore, I use Azure CLI in the Powershell
