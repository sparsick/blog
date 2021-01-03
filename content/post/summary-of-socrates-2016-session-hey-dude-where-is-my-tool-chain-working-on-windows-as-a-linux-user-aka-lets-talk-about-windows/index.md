---
title: 'Summary of SoCraTes 2016 Session "Hey dude, where is my tool chain?" - Working on Windows as a Linux'
date: 2016-09-20
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
  - Conference
  - Linux
  - Windows
tags:
  - conference
  - linux
  - socrates
  - tool tip


# comment: false # Disable comment if false.
---

This year on the conference SoCraTes I hosted a session for the first time. It was about working on a Windows system from the perspective of a Linux user.  A big thank to [@ndrssmn](https://twitter.com/ndrssmn), who motivated to host this session.

[@yooogan](https://twitter.com/yooogan) was so nice to summarize the session in the [SoCraTes wiki](https://www.socrates-conference.de/wiki/2016/session_windows_tools) (big thank for that).  But the wiki page is only accessible for SoCraTes participants, so we decided that I republish it on my blog. Enjoy it.

**Console/Shell**

*   [Babun](https://babun.github.io/) - Based on Cygwin, includes a CLI package manager (`pact` - like `apt`, `yum`, …) and preconfigured `oh-my-zsh` as shell
*   [ConEmu](https://conemu.github.io/) - feature rich console emulator with tabs
*   Console2 ([original](https://github.com/cbucher/console))/([modified fork](https://github.com/cbucher/console)) - console emulator, multi tabs, configurable mouse behavior
*   [PuTTY](http://www.putty.org/) - `ssh` client (when you don't have Babun/Cygwin anyway)

**File Management**

*   [FreeCommander](http://freecommander.com/en/summary/) - Replacement for Windows Explorer / TotalCommander
*   [SourceTree](https://www.sourcetreeapp.com/) - graphical Git and Mercurial client
*   [WinSCP](https://winscp.net/) - graphical `scp`
*   [FreeFileSync](http://www.freefilesync.org/) - multi-platform file synchronization

**Text Editors**

*   [Notepad++](https://notepad-plus-plus.org/) - all-purpose editor, syntax highlighting, file monitoring (`tail -f`)
*   [Atom](https://atom.io/) - editor; same settings in all your environments

**Diff/Merge**

*   [kdiff3](http://kdiff3.sourceforge.net/) - nice directory compare mode
*   [p4merge](https://www.perforce.com/downloads/integrations)
*   [Meld](http://meldmerge.org/)
*   [BeyondCompare](http://www.scootersoftware.com/) - commercial, but powerful (3-way merge, plugins to compare .doc/.pdf/etc.)

**Searching**

*   [Agent Ransack](https://www.mythicsoft.com/agentransack) - GUI searcher
*   [`ack`](http://beyondgrep.com/) - multi-platform (single Perl script) grep replacement - faster and with good defaults for source trees
*   [`ag` - The Silver Searcher](http://geoff.greer.fm/ag/) - even faster than `ack`

**Viewers**

*   [SumatraPDF](http://www.sumatrapdfreader.org/free-pdf-reader.html) - lightweight PDF viewer

**Documentation**

*   [Zim](http://zim-wiki.org/) - Organize notes, saves to plain text
*   [Greenshot](http://getgreenshot.org/de/) - Screenshots, including obfuscation / comments, for documentation, connects to JIRA
*   [yEd](https://www.yworks.com/products/yed) - multi-platform (Win/Linux/MacOS) graph editor, extensible palette, useful pictograms
*   [Paint.NET](http://www.getpaint.net/index.html) - free image editor
*   [GIMP](https://www.gimp.org/) - Swiss army knife for images

**System**

*   [Process Explorer](https://technet.microsoft.com/en-us/sysinternals/bb896653.aspx) - better task manager
*   [Autoruns](https://technet.microsoft.com/en-us/sysinternals/bb963902.aspx) - manage automatically started programs
*   [Rapid Environment Editor](http://www.rapidee.com) - environment variables editor

**Disk Usage**

*   [RidNacs](http://www.splashsoft.de/Freeware/ridnacs-disk-space-usage-analyzer.html) - graphical du
*   [WinDirStat](https://windirstat.info/) - even more colorful graphical du
*   [`ncdu`](https://dev.yorhel.nl/ncdu/scr) - CLI, can be installed from Cygwin/Babun

**Keyboard**

*   [WinCompose](https://github.com/SamHocevar/wincompose) - a (X.org/X11 like) compose key for Windows - type `äöëß€«»←↑↓→¡☺♥…` like a boss!
*   [SharpKeys](http://www.randyrants.com/category/sharpkeys/) - remap keyboard: `CapsLock` → `Ctrl`, `~` → `Esc`, etc.
*   [AutoHotkey](https://www.autohotkey.com/) - very sophisticated keyboard macros / automation - full-fledged scripting language

**Clipboard**

*   [PureText](http://stevemiller.net/puretext/) - remove formatting from pasted text
*   [Ditto](http://ditto-cp.sourceforge.net/) - clipboard manager

Pictures (taken from [Twitter](https://twitter.com/SandraParsick/status/769545705614675968)):

![Let's talk about Windows, pt. 1](https://pbs.twimg.com/media/Cq35uj_WAAAZN3P.jpg:large "Let's talk about Windows, pt. 1")
![Let's talk about Windows, pt. 2](https://pbs.twimg.com/media/Cq35vLvWIAEBmEq.jpg:large "Let's talk about Windows, pt. 2")
![Let's talk about Windows, pt. 3](https://pbs.twimg.com/media/Cq35u2gWcAALi4m.jpg:large "Let's talk about Windows, pt. 3")
