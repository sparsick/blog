---
title: 'Pimp My Git - Manage Different Git Identities'
date: 2017-07-24
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
  - identities
  - persona
  - git

# comment: false # Disable comment if false.
---

I usually work on different Git projects that need different Git identities. My work flow for new repositories was

1.  Clone new repository.
2.  Go to cloned repository.
3.  If it is necessary to change the Git identity, call a shell script that runs `git config user.name "Sandra Parsick"; git config user.email sparsick@web.de`

I was never happy with this solution, but it works. Fortunately, a tweet of [@BenediktRitter](https://twitter.com/BenediktRitter) and one of [@wosc](https://twitter.com/wosc) suggest two alternatives to my method.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr"><a href="https://twitter.com/hashtag/TIL?src=hash&amp;ref_src=twsrc%5Etfw">#TIL</a>: you can configure multiple git identities. Awesome when you switch between work and private projects! <a href="https://t.co/0WNSCdYbOy">https://t.co/0WNSCdYbOy</a></p>&mdash; Benedikt Ritter (@BenediktRitter) <a href="https://twitter.com/BenediktRitter/status/882190899073122304?ref_src=twsrc%5Etfw">July 4, 2017</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none"><p lang="en" dir="ltr"><a href="https://twitter.com/SandraParsick?ref_src=twsrc%5Etfw">@SandraParsick</a> or you could use an explicit switcher if you prefer <a href="https://t.co/KDm8iPxoxe">https://t.co/KDm8iPxoxe</a> (shameless plug ;)</p>&mdash; Wolfgang Schnerring (@wosc) <a href="https://twitter.com/wosc/status/882618552771129344?ref_src=twsrc%5Etfw">July 5, 2017</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

The first method bases on the Git feature _Conditional Includes_ (required Git Version at least 2.13). The idea is that you define a default Git identity and separate Git identities per specific directory. That means, every repository, that is cloned beneath one of the specific directory, has automatically its specified Git identities. The second method bases on a Python script, that is inspired by the Mercurial extension hg-persona. The idea is that you can individually set a Git identity per Git repository. It is an one command replacement for the `git config user.*` command serie. In the next two sections I'd like to summarize how to set up and how to use these two methods. I have tested it on a Debian-based system. Let's start with the first one.

Summarize Git identity for several Git repositories
---------------------------------------------------

As above described, this method bases on the Git feature _Conditional Includes_. Therefore, ensure you have installed your Git client in at least version 2.13 . Assume, we want to have two Git identities, one for Github and one for work. Therefore, create two `.gitconfig` files in your home folder.

```bash
touch ~/.gitconfig_github
touch ~/.gitconfig_work
```

 Then add the specific Git identity in respective `.gitconfig` files.

```bash
~/.gitconfig_github

[user]
   name = YourNameForGithub
   email = name@forgithub.com

~/.gitconfig_work

[user]
   name = YourNameForWork
   email = name@forwork.com
```
The next step is to add these two `.gitconfig` files to our global one and to specify when to use them.

```bash
~/.gitconfig

[user]
   name = defaultName
   email = default@email.com

[includeIf "gitdir:~/workspace_work/"]
   path = .gitconfig_work

[includeIf "gitdir:~/workspace_github/"]
   path = .gitconfig_github
```
Now, every repository, that is cloned beneath `~/workspace_work/`, has automatically the Git identity for Work (`.gitconfig_work`) and every repository, that is cloned beneath `~/workspace_github/`, has automatically the Git identity for Github (`.gitconfig_github`). Otherwise, the default Git identity is used.

Setting Git identity individually per Git repository
----------------------------------------------------

For the second method, you have to install ws.git-persona from PyPI.

```bash
sudo apt-get install pip # if PyPI isn't install
pip install ws.git-persona
```
Then, open your global `~/.gitconfig` and add your personas. In our cases, we add two personas, one for Github and one for work.


```bash
~/.gitconfig

[persona]
  github = YourNameForGithub <name@forgithub.com>
  work = YourNameForWork <name@forwork.com>
```
In the next step, we want to switch our Git identity in a Git repository. This is now possible with the command `git-persona`. In the following example we switch to the identity for Github and then to the identity for work.

```bash
> git-persona -n github
Setting user.name="YourNameForGithub", user.email="name@forgithub.com"
> git config user.name
YourNameForGithub
> git config user.email
name@forgithub.com
> git-persona -n work
Setting user.name="YourNameForWork", user.email="name@forwork.com"
> git config user.email
name@forwork.com
> git config user.name
YourNameForWork
```

If you have other methods to manage different Git identities, let me know it and write a comment.  

Links
-----

1.  [Blog post](https://dev.to/maxlmator/maintaining-different-git-identities) about Git feature "Conditional Includes".
2.  [Github repository](https://github.com/wosc/git-persona) of git-personas.
