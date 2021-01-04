---
title: "Apache2 as Reverse Proxy for NPM Registry Proxies in Sonatype Nexus 3"
date: 2018-04-29
description: "Article description." # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
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
  - Continuous Integration
  - Nexus
  - Node.js
tags:
  - apache2
  - configuration
  - npm
  - npm registry
  - proxy
# comment: false # Disable comment if false.
---

I use a NPM registry proxy in Sonatype Nexus 3 behind an Apache2 as reverse proxy. With the "standard" Apache2 VirtualHost configuration
```shell
<VirtualHost:80>
  ProxyRequests Off
  <Proxy *>
    Order deny,allow
    Allow from all
  </Proxy>
  ProxyPass / http://localhost:8081/
  ProxyPassReverse / http://localhost:8081/
</VirtualHost:80>
```
I got following failure when I tried to install the dependency `@sinonjs/formatio`.

```shell
$ yarn add @sinonjs/formatio --verbose
 yarn add v1.3.2
 warning package.json: No license field
 verbose 0.337 Checking for configuration file "/home/sparsick/dev/workspace/yarn-test-module/.npmrc".
 verbose 0.337 Checking for configuration file "/home/sparsick/.npmrc".
 verbose 0.337 Found configuration file "/home/sparsick/.npmrc".
 verbose 0.337 Checking for configuration file "/usr/etc/npmrc".
 verbose 0.338 Found configuration file "/usr/etc/npmrc".
 verbose 0.338 Checking for configuration file "/home/sparsick/dev/workspace/yarn-test-module/.npmrc".
 verbose 0.338 Checking for configuration file "/home/sparsick/dev/workspace/.npmrc".
 verbose 0.338 Checking for configuration file "/home/sparsick/dev/.npmrc".
 verbose 0.338 Checking for configuration file "/home/sparsick/.npmrc".
 verbose 0.338 Found configuration file "/home/sparsick/.npmrc".
 verbose 0.338 Checking for configuration file "/home/.npmrc".
 verbose 0.341 Checking for configuration file "/home/sparsick/dev/workspace/yarn-test-module/.yarnrc".
 verbose 0.342 Found configuration file "/home/sparsick/dev/workspace/yarn-test-module/.yarnrc".
 verbose 0.343 Checking for configuration file "/home/sparsick/.yarnrc".
 verbose 0.344 Found configuration file "/home/sparsick/.yarnrc".
 verbose 0.344 Checking for configuration file "/usr/etc/yarnrc".
 verbose 0.344 Checking for configuration file "/home/sparsick/dev/workspace/yarn-test-module/.yarnrc".
 verbose 0.345 Found configuration file "/home/sparsick/dev/workspace/yarn-test-module/.yarnrc".
 verbose 0.345 Checking for configuration file "/home/sparsick/dev/workspace/.yarnrc".
 verbose 0.345 Checking for configuration file "/home/sparsick/dev/.yarnrc".
 verbose 0.345 Checking for configuration file "/home/sparsick/.yarnrc".
 verbose 0.345 Found configuration file "/home/sparsick/.yarnrc".
 verbose 0.345 Checking for configuration file "/home/.yarnrc".
 verbose 0.347 current time: 2018-02-27T08:04:43.357Z warning yarn-test-module: No license field \[1/4\] Resolving packages...
 verbose 0.45 Performing "GET" request to "http://mycompany/repository/npm-public/@sinonjs%2fformatio".
 verbose 0.55 Request "http://mycompany/repository/npm-public/@sinonjs%2fformatio" finished with status code 404.
 verbose 0.551 Error: Couldn't find package "@sinonjs/formatio" on the "npm" registry. at /usr/lib/node\_modules/yarn/lib/cli.js:49061:15 at Generator.next (<anonymous>) at step (/usr/lib/node\_modules/yarn/lib/cli.js:92:30) at /usr/lib/node\_modules/yarn/lib/cli.js:103:13 at <anonymous> at process.\_tickCallback (internal/process/next\_tick.js:188:7) error Couldn't find package "@sinonjs/formatio" on the "npm" registry. info Visit https://yarnpkg.com/en/docs/cli/add for documentation about this command.
 ```
 The problem is that Apache2 canonicalizes URLs as default. So I have to configure Apache2 to not canonicalize URLs and additionally, I have to allow encoded slashes:

```shell
<VirtualHost:80>

  ProxyRequests Off
  <Proxy *>
    Order deny,allow
    Allow from all
  </Proxy>
  ProxyPass / http://localhost:8081/ nocanon
  ProxyPassReverse / http://localhost:8081/

  AllowEncodedSlashes NoDecode

</VirtualHost:80>
```

With the above Apache2 Virtualhost configuration I could install my dependency via the NPM registry proxy.

```shell
$ yarn add @sinonjs/formatio

yarn add v1.3.2
warning package.json: No license field
info No lockfile found.
warning yarn-test-module: No license field
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
success Saved lockfile.
success Saved 2 new dependencies.
├─ @sinonjs/formatio@2.0.0
└─ samsam@1.3.0
warning yarn-test-module: No license field
Done in 0.60s.
```
Big thanks to Sonatype support team, that gave me this advice.
