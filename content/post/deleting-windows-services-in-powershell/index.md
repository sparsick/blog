---
title: 'Deleting Windows Services In PowerShell'
date: 2016-05-11
#description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - PowerShell
  - Windows

# comment: false # Disable comment if false.
---


For deleting a Windows service in Windows CMD, you have to call

```cmd
sc delete "Service Name"
```

This doesn't work in PowerShell, because `sc` is an alias for the cmdlet `Set-Content` in PowerShell. Deleting Windows service in PowerShell can be done by calling

```powershell
sc.exe delete "Service Name"
```
Note, that PowerShell has to be run as administrator.
