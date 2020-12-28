---
title: 'Commons VFS, SSHJ and JSch in Comparison'
date: 2015-07-30
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
  - Java
tags:
  - commons-vfs
  - Java
  - jsch
  - sftp
  - ssh
  - sshj


# comment: false # Disable comment if false.
---
Some weeks ago I evaluated some SSH libraries for Java. The main requirements to them are file transferring and file operations on a remote machine. Therefore, it exists a network protocol based on SSH, SSH File Transfer Protocol (or SFTP). So I needed a SSH library that supports SFTP. A research shows that it exits many SSH libraries for Java. I reduce the number of libraries to three for the comparison. I choose JSch, SSHJ and Apache's Commons VFS for a deeper look. All of them support SFTP. JSch seems to be the de-facto standard for Java. SSHJ is a newer library. Its goal is to have a clear Java API for SSH. The goal of Commons VFS is to have a clear API for virtual file systems and SFTP is one of the supported protocol. Under the hood it uses JSch for the SFTP protocol. The libraries should cover following requirements:

*   client authentication over password
*   client authentication over public key
*   server authentication
*   upload files from local host over SFTP
*   download files to local host over SFTP
*   file operations on the remote host like move, delete, list all children of a given folder (filtering after type like file or folder) over SFTP
*   execute plain shell commands

Lets have a deeper look how the three libraries cover the requirements.

Client Authentication
---------------------

All three libraries supports both required authentication methods. SSHJ has the clearest API for authentication (`SSHClient.authUserPass(), SSHClient.authUserPublicKey()`).

```java
SSHClient sshClient= new SSHClient();
sshClient.connect(host);

// only for public key authentication
sshClient.authPublickey("user", "location to private key file");

// only for password authentication
sshClient.authPassword("user", "password");
```
In Commons VFS the authentication configuration depends which kind of authentication should be used. For the public key authentication, the private key has to set in the `FileSystemOption` and the user name is a part of the connection url. For the password authentication, user name and password is a part of the connection url.

```java
StandardFileSystemManager fileSystemManager = new StandardFileSystemManager();
fileSystemManager.init();

// only for public key authentication
SftpFileSystemConfigBuilder sftpConfigBuilder = SftpFileSystemConfigBuilder.getInstance();
FileSystemOptions opts = new FileSystemOptions();
sftpConfigBuilder.setIdentities(opts, new File[]{privateKey.toFile()});
String connectionUrl = String.format("sftp://%s@%s", user, host);

// only for password authentication
String connectionUrl = String.format("sftp://%s:%s@%s", user, password, host);

// Connection set-up
FileObject remoteRootDirectory = fileSystemManager.resolveFile(connectionUrl, connectionOptions);
```
The authentication configuration in JSch is similar to Commons VFS. It depends which kind of authentication should be used. The private key for the public key authentication has to be configured in the `JSch` object and the password for the password authentication has to be set in the `Session` object. For both, the user name is set, when the `JSch` object gets the `Session` object.

```java
JSch sshClient = new JSch();

// only for public key authentication
sshClient.addIdentity("location to private key file");

session = sshClient.getSession(user, host);

// only for password authentication
session.setPassword(password);

session.connect();
```

Server Authentication
---------------------

All three libraries supports server authentication. In SSHJ the server authentication can be enabled with `SSHClient.loadKnownHost()`. It is possible to  add an own location of `known_host` file or it is used the default location that depends on the using platform.

```java
SSHClient sshClient = new SSHClient();
sshClient.loadKnownHosts(); // or sshClient.loadKnownHosts(knownHosts.toFile());
sshClient.connect(host);
```
In Commons VFS the server authentication configuration is also a part of the `FileSystemOption` like the public key authentication. There, the location of the `known_hosts` file can be set.

```java
SftpFileSystemConfigBuilder sftpConfigBuilder = SftpFileSystemConfigBuilder.getInstance();
FileSystemOptions opts = new FileSystemOptions();
sftpConfigBuilder.setKnownHosts(opts, new File("location of the known_hosts file"));
```
In JSch it exists two possibilities to configure the server authentication. One possibility is to use the `OpenSSHConfig` (see [JSch example for OpenSSHConfig](http://www.jcraft.com/jsch/examples/OpenSSHConfig.java.html)). The another possibility is easier. The location of the `known_hosts` file can be set directly in `JSch` object.

```java
JSch sshClient = new JSch();
sshClient.setKnownHosts("location of known-hosts file");
```

Upload/download Files Over SFTP
-------------------------------

All three libraries supports uploads and downloads files over SFTP. SSHJ has very clear API for these operations. The `SSHClient` object creates a `SFTPClient` object. This object is responsible for the upload (`SFTPClient.put()`) and for the download (`SFTPClient.get()`).

```java
SSHClient sshClient = new SSHClient();
// ... connection

try (SFTPClient sftpClient = sshClient.newSFTPClient()) {
  // download
  sftpClient.get(remotePath, new FileSystemFile(local.toFile()));
  // upload
  sftpClient.put(new FileSystemFile(local.toFile()), remotePath);
}
```
In Commons VFS the upload and download files is abstracted as operation on a file system. So both are represented by the `copyFrom` method of a `FileObject` object. Upload is a `copyFrom` operation on a `RemoteFile` object and download is a `copyFrom` operation on a `LocalFile.`

```java
StandardFileSystemManager fileSystemManager = new StandardFileSystemManager();
// ... configuration
remoteRootDirectory = fileSystemManager.resolveFile(connectionUrl, connectionOptions);

LocalFile localFileObject = (LocalFile) fileSystemManager.resolveFile(local.toUri().toString());
FileObject remoteFileObject = remoteRootDirectory.resolveFile(remotePath);
try {
  // download
  localFileObject.copyFrom(remoteFileObject, new AllFileSelector());

  // upload
  remoteFileObject.copyFrom(localFileObject, new AllFileSelector());
} finally {
  localFileObject.close();
  remoteFileObject.close();
}
```
JSch also supports a SFTPClient. In JSch it is called ChannelSFTP. It has two method for download (`ChannelSFTP.get()`) and upload (`ChannelSFTP.put()`).


```java
// here: creation and configuration of session

ChannelSftp sftpChannel = null;
try {
  sftpChannel = (ChannelSftp) session.openChannel("sftp");
  sftpChannel.connect();

  // download
  InputStream inputStream = sftpChannel.get(remotePath);
  Files.copy(inputStream, localPath);

  // upload
  OutputStream outputStream = sftpChannel.put(remotePath);
  Files.copy(locaPathl, outputStream);
} catch (SftpException | JSchException ex) {
  throw new IOException(ex);
} finally {
  if (sftpChannel != null) {
    sftpChannel.disconnect();
  }
}
```


Execute Shell Commands
----------------------

Only Commons VFS doesn't support executing plain shell commands. In SSHJ it is a two-liner. The `SshClient` starts a new `Session` object. This object executes the shell command. It is very intuitive.

```java
// creation and configuration of sshClient

try (Session session = sshClient.startSession()) {
  session.exec("ls");
}
```

In Jsch the `ChannelExec` is responsible for executing shell commands over SSH. At first the command is set in the channel and then the channel has to be started. It isn't so intuitive than in SSHJ.

```java
// here: creation and configuration of session object

ChannelExec execChannel = null;
try {
  execChannel = (ChannelExec) session.openChannel("exec");
  execChannel.connect();
  execChannel.setCommand(command);
  execChannel.start();
} catch (JSchException ex) {
  throw new IOException(ex);
} finally {
  if (execChannel != null) {
    execChannel.disconnect();
  }
}
```

File Operations On the Remote Hosts
-----------------------------------

All libraries supports more or less ideal file operations over SFTP on remote machines. In SSHJ `SFTPClient` has also methods for file operations. The names of the methods are the same as the file operations on a Linux system. The following code snippet shows how to delete a file.

```java
//here: creation and configuration of sshClient

try (SFTPClient sftpClient = sshClient.newSFTPClient()) {
  sftpClient.rm(remotePath);
}
```
Commons VFS's core functionality is file operations. The usage takes getting used to. A file object has to be resolve and the file operations can be done on it.

```java
// here: creation and configuration of remoteRootDirectory

FileObject remoteFileObject = remoteRootDirectory.resolveFile(remotePath);
try {
  remoteFileObject.delete();
} finally {
  remoteFileObject.close();
}
```

JSch's SFTPClient `ChannelSFTP` has also method for file operations. The mostly file operations are supported by this channel. For e.g. the file copy operation on the remote machine has to be done by plain shell commands over the `ChannelExec`.

```java
// here: creation and configuration of session
ChannelSftp sftpChannel = null;
try {
  sftpChannel = (ChannelSftp) session.openChannel("sftp");
  sftpChannel.connect();
  sftpChannel.rm(remotePath);
} catch (SftpException | JSchException ex) {
  throw new IOException(ex);
} finally {
  if (sftpChannel != null) {
    sftpChannel.disconnect();
  }
}
```

Conclusion
----------

After this comparison I have two favourites, SSHJ and Commons VFS. SSHJ has a very clear API and I would choose it if I need a common SSH client or file operation support over SFTP is sufficient. I would choose Commons VFS if I have file operation over many file system protocols or a common SSH client is not needed. For the case, that I need both, I could use JSch directly to execute commands over SSH. The API of Commons VFS takes getting used to. But after understanding the concept behind, the usage of the API is straightforward. The whole source code examples of this comparison are hosted on [Github](https://github.com/sparsick/comparison-java-ssh-libs).

Useful Links
------------

1.  [SSHJ homepage](https://github.com/hierynomus/sshj)
2.  [JSch homepage](http://www.jcraft.com/jsch/)
3.  [Commons-vfs homepage](https://commons.apache.org/proper/commons-vfs/)
4.  [Wikipedia page about SFTP](https://en.wikipedia.org/wiki/SSH_File_Transfer_Protocol)
5.  [Source Code of this comparison on Github](https://github.com/sparsick/comparison-java-ssh-libs)
