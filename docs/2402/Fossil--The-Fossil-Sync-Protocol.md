<!--yml
category: 未分类
date: 2024-05-29 13:19:42
-->

# Fossil: The Fossil Sync Protocol

> 来源：[https://fossil-scm.org/home/doc/trunk/www/sync.wiki](https://fossil-scm.org/home/doc/trunk/www/sync.wiki)

 This document describes the wire protocol used to synchronize content between two Fossil repositories.

## 1.0 Overview

The global state of a fossil repository consists of an unordered [collection of artifacts](https://fossil-scm.org/home/doc/trunk/www/fileformat.wiki). Each artifact is identified by a cryptographic hash of its content, expressed as a lower-case hexadecimal string. Synchronization is the process of sharing artifacts between repositories so that all repositories have copies of all artifacts. Because artifacts are unordered, the order in which artifacts are received is unimportant. It is assumed that the hash names of artifacts are unique - that every artifact has a different hash. To a first approximation, synchronization proceeds by sharing lists of hashes for available artifacts, then sharing the content of artifacts whose names are missing from one side or the other of the connection. In practice, a repository might contain millions of artifacts. The list of hash names for this many artifacts can be large. So optimizations are employed that usually reduce the number of hashes that need to be shared to a few dozen.

Each repository also has local state. The local state determines the web-page formatting preferences, authorized users, ticket formats, and similar information that varies from one repository to another. The local state is not usually transferred during a sync. Except, some local state is transferred during a [clone](https://fossil-scm.org/home/help?cmd=clone) in order to initialize the local state of the new repository. Also, an administrator can sync local state using the [config push](https://fossil-scm.org/home/help?cmd=configuration) and [config pull](https://fossil-scm.org/home/help?cmd=configuration) commands.

### 1.1 Conflict-Free Replicated Datatypes

The "bag of artifacts" data model used by Fossil is apparently an implementation of a particular [Conflict-Free Replicated Datatype (CRDT)](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) called a "G-Set" or "Grow-only Set". The academic literature on CRDTs only began to appear in about 2011, and Fossil predates that research by at least 4 years. But it is nice to know that theorists have now proven that the underlying data model of Fossil can provide strongly-consistent replicas using only peer-to-peer communication and without any kind of central authority.

If you are already familiar with CRDTs and were wondering if Fossil used them, the answer is "yes". We just don't call them by that name.

## 2.0 Transport

All communication between client and server is via HTTP requests. The server is listening for incoming HTTP requests. The client issues one or more HTTP requests and receives replies for each request.

The server might be running as an independent server using the ["fossil server" command](https://fossil-scm.org/home/help?cmd=server), or it might be launched from inetd or xinetd using the [/help?cmd=http](https://fossil-scm.org/home/wiki?name=%22fossil+http%22+command). Or the server might be [launched from CGI](https://fossil-scm.org/home/doc/trunk/www/server/any/cgi.md) or from [SCGI](https://fossil-scm.org/home/doc/trunk/www/server/any/scgi.md). (See "[How To Configure A Fossil Server](https://fossil-scm.org/home/doc/trunk/www/server/)" for details.) The specifics of how the server listens for incoming HTTP requests is immaterial to this protocol. The important point is that the server is listening for requests and the client is the issuer of the requests.

A single [push](https://fossil-scm.org/home/help?cmd=push), [pull](https://fossil-scm.org/home/help?cmd=pull), or [sync](https://fossil-scm.org/home/help?cmd=sync) might involve multiple HTTP requests. The client maintains state between all requests. But on the server side, each request is independent. The server does not preserve any information about the client from one request to the next.

Note: Throughout this article, we use the terms "server" and "client" to represent the listener and initiator of the interaction, respectively. Nothing in this protocol requires that the server actually be a back-room processor housed in a datacenter, nor does the client need to be a desktop or handheld device. For the purposes of this article "client" simply means the repository that initiates the conversation and "server" is the repository that responds. Nothing more.

#### 2.0.1 HTTPS Transport

HTTPS differs from HTTP only in that the HTTPS protocol is encrypted as it travels over the wire. The underlying protocol is the same. This document describes only the underlying, unencrypted messages that go client to server and back to client. Whether or not those messages are encrypted does not come into play in this document.

Fossil includes built-in [support for HTTPS encryption](https://fossil-scm.org/home/doc/trunk/www/ssl-server.md) in both client and server.

#### 2.0.2 SSH Transport

When doing a sync using an "`ssh:…`" URL, the same HTTP transport protocol is used. Fossil simply uses [ssh](https://en.wikipedia.org/wiki/Secure_Shell) to start an instance of the [fossil test-http](https://fossil-scm.org/home/help?cmd=test-http) command running on the remote machine. It then sends HTTP requests and gets back HTTP replies over the SSH connection, rather than sending and receiving over an internet socket. To see the specific "ssh" command that the Fossil client runs in order to set up a connection, add either of the the "--httptrace" or "--sshtrace" options to the "fossil sync" command line.

This method is dependent on the remote `PATH` set by the SSH daemon, which may not be the same as your interactive shell's `PATH` on that same server. It is common to find `$HOME/bin` in the latter but not the former, for instance, leading to failures to sync over `ssh:…` URLs when you install the `fossil` binary in a nonstandard location, as with

```
./configure --prefix=$HOME && make install
```

The simpler of the two solutions to this problem is to install Fossil where sshd expects to find it, but when that isn't an option, you can instead give a URL like this:

```
fossil clone ssh://myserver.example.com/path/to/repo.fossil?fossil=/home/me/bin/fossil
```

That gives the local Fossil instance the absolute path to the binary on the remote machine for use when calling that Fossil instance through the SSH tunnel.

#### 2.0.3 FILE Transport

When doing a sync using a "file:..." URL, the same HTTP protocol is still used. But instead of sending each HTTP request over a socket or via SSH, the HTTP request is written into a temporary file. The client then invokes the [fossil http](https://fossil-scm.org/home/help?cmd=http) command in a subprocess to process the request and and generate a reply. The client then reads the HTTP reply out of a temporary file on disk, and deletes the two temporary files. To see the specific "fossil http" command that is run in order to implement the "file:" transport, add the "--httptrace" option to the "fossil sync" command.

### 2.1 Server Identification

The server is identified by a URL argument that accompanies the push, pull, or sync command on the client. (As a convenience to users, the URL can be omitted on the client command and the same URL from the most recent push, pull, or sync will be reused. This saves typing in the common case where the client does multiple syncs to the same server.)

The client modifies the URL by appending the method name "**/xfer**" to the end. For example, if the URL specified on the client command line is

```
https://fossil-scm.org/fossil
```

Then the URL that is really used to do the synchronization will be:

```
https://fossil-scm.org/fossil/xfer
```

### 2.2 HTTP Request Format

The client always sends a POST request to the server. The general format of the POST request is as follows:

```
POST /fossil/xfer HTTP/1.0
Host: fossil-scm.hwaci.com:80
Content-Type: application/x-fossil
Content-Length: 4216

```

```
*content...*
```

In the example above, the pathname given after the POST keyword on the first line is a copy of the URL pathname. The Host: parameter is also taken from the URL. The content type is always either "application/x-fossil" or "application/x-fossil-debug". The "x-fossil" content type is the default. The only difference is that "x-fossil" content is compressed using zlib whereas "x-fossil-debug" is sent uncompressed.

A typical reply from the server might look something like this:

```
HTTP/1.0 200 OK
Date: Mon, 10 Sep 2007 12:21:01 GMT
Connection: close
Cache-control: private
Content-Type: application/x-fossil; charset=US-ASCII
Content-Length: 265

```

```
*content...*
```

The content type of the reply is always the same as the content type of the request.

## 3.0 Fossil Synchronization Content

A synchronization request between a client and server consists of one or more HTTP requests as described in the previous section. This section details the "x-fossil" content type.

### 3.1 Line-oriented Format

The x-fossil content type consists of zero or more "cards". Cards are separated by the newline character ("\n"). Leading and trailing whitespace on a card is ignored. Blank cards are ignored.

Each card is divided into zero or more space separated tokens. The first token on each card is the operator. Subsequent tokens are arguments. The set of operators understood by servers is slightly different from the operators understood by clients, though the two are very similar.

### 3.2 Login Cards

Every message from client to server begins with one or more login cards. Each login card has the following format:

```
login  *userid  nonce  signature*
```

The userid is the name of the user that is requesting service from the server. The nonce is the SHA1 hash of the remainder of the message - all text that follows the newline character that terminates the login card. The signature is the SHA1 hash of the concatenation of the nonce and the users password.

For each login card, the server looks up the user and verifies that the nonce matches the SHA1 hash of the remainder of the message. It then checks the signature hash to make sure the signature matches. If everything checks out, then the client is granted all privileges of the specified user.

Privileges are cumulative. There can be multiple successful login cards. The session privilege is the union of all privileges from all login cards.

### 3.3 File Cards

Artifacts are transferred using either "file" cards, or "cfile" or "uvfile" cards. The name "file" card comes from the fact that most artifacts correspond to files that are under version control. The "cfile" name is an abbreviation for "compressed file". The "uvfile" name is an abbreviation for "unversioned file".

#### 3.3.1 Ordinary File Cards

For sync protocols, artifacts are transferred using "file" cards. File cards come in two different formats depending on whether the artifact is sent directly or as a [delta](https://fossil-scm.org/home/doc/trunk/www/delta_format.wiki) from some other artifact.

```
file *artifact-id size* \n *content*
file *artifact-id delta-artifact-id size* \n *content*

```

File cards are followed by in-line "payload" data. The content of the artifact or the artifact delta is the first *size* bytes of the x-fossil content that immediately follow the newline that terminates the file card.

The first argument of a file card is the ID of the artifact that is being transferred. The artifact ID is the lower-case hexadecimal representation of the name hash for the artifact. The last argument of the file card is the number of bytes of payload that immediately follow the file card. If the file card has only two arguments, that means the payload is the complete content of the artifact. If the file card has three arguments, then the payload is a [delta](https://fossil-scm.org/home/doc/trunk/www/delta_format.wiki) and the second argument is the ID of another artifact that is the source of the delta.

File cards are sent in both directions: client to server and server to client. A delta might be sent before the source of the delta, so both client and server should remember deltas and be able to apply them when their source arrives.

#### 3.3.2 Compressed File Cards

A client that sends a clone protocol version "3" or greater will receive artifacts as "cfile" cards while cloning. This card was introduced to improve the speed of the transfer of content by sending the compressed artifact directly from the server database to the client.

Compressed File cards are similar to File cards, sharing the same in-line "payload" data characteristics and also the same treatment of direct content or delta content. Cfile cards come in two different formats depending on whether the artifact is sent directly or as a delta from some other artifact.

```
cfile *artifact-id usize csize* \n *content*
cfile *artifact-id delta-artifact-id usize csize* \n *content*

```

The first argument of the cfile card is the ID of the artifact that is being transferred. The artifact ID is the lower-case hexadecimal representation of the name hash for the artifact. The second argument of the cfile card is the original size in bytes of the artifact. The last argument of the cfile card is the number of compressed bytes of payload that immediately follow the cfile card. If the cfile card has only three arguments, that means the payload is the complete content of the artifact. If the cfile card has four arguments, then the payload is a delta and the second argument is the ID of another artifact that is the source of the delta and the third argument is the original size of the delta artifact.

Unlike file cards, cfile cards are only sent in one direction during a clone from server to client for clone protocol version "3" or greater.

#### 3.3.3 Private artifacts

"Private" content consist of artifacts that are not normally synced. However, private content will be synced when the the [fossil sync](https://fossil-scm.org/home/help?cmd=sync) command includes the "--private" option.

Private content is marked by a "private" card:

```
private
```

The private card has no arguments and must directly precede a file card that contains the private content.

#### 3.3.4 Unversioned File Cards

Unversioned content is sent in both directions (client to server and server to client) using "uvfile" cards in the following format:

```
uvfile *name mtime hash size flags* \n *content*
```

The *name* field is the name of the unversioned file. The *mtime* is the last modification time of the file in seconds since 1970\. The *hash* field is the hash of the content for the unversioned file, or "**-**" for deleted content. The *size* field is the (uncompressed) size of the content in bytes. The *flags* field is an integer which is interpreted as an array of bits. The 0x0004 bit of *flags* indicates that the *content* is to be omitted. The content might be omitted if it is too large to transmit, or if the sender merely wants to update the modification time of the file without changing the files content. The *content* is the (uncompressed) content of the file.

The receiver should only accept the uvfile card if the hash and size match the content and if the mtime is newer than any existing instance of the same file held by the receiver. The sender will not normally transmit a uvfile card unless all these constraints are true, but the receiver should double-check.

A server will only accept uvfile cards if the login user has the "y" write-unversioned permission.

Servers send uvfile cards in response to uvgimme cards received from the client. Clients send uvfile cards when they determine that the server needs the content based on uvigot cards previously received from the server.

### 3.4 Push and Pull Cards

Among the first cards in a client-to-server message are the push and pull cards. The push card tells the server that the client is pushing content. The pull card tells the server that the client wants to pull content. In the event of a sync, both cards are sent. The format is as follows:

```
push *servercode projectcode*
pull *servercode projectcode*

```

The *servercode* argument is the repository ID for the client. The *projectcode* is the identifier of the software project that the client repository contains. The projectcode for the client and server must match in order for the transaction to proceed.

The server will also send a push card back to the client during a clone. This is how the client determines what project code to put in the new repository it is constructing.

The *servercode* argument is currently unused.

### 3.5 Clone Cards

A clone card works like a pull card in that it is sent from client to server in order to tell the server that the client wants to pull content. The clone card comes in two formats. Older clients use the no-argument format and newer clients use the two-argument format.

```
clone
clone *protocol-version sequence-number*

```

#### 3.5.1 Protocol 3

The latest clients send a two-argument clone message with a protocol version of "3". (Future versions of Fossil might use larger protocol version numbers.) Version "3" of the protocol enhanced version "2" by introducing the "cfile" card which is intended to speed up clone operations. Instead of sending "file" cards, the server will send "cfile" cards

#### 3.5.2 Protocol 2

The sequence-number sent is the number of artifacts received so far. For the first clone message, the sequence number is 0\. The server will respond by sending file cards for some number of artifacts up to the maximum message size.

The server will also send a single "clone_seqno" card to the client so that the client can know where the server left off.

```
clone_seqno  *sequence-number*

```

The clone message in subsequent HTTP requests for the same clone operation will use the sequence-number from the clone_seqno of the previous reply.

In response to an initial clone message, the server also sends the client a push message so that the client can discover the projectcode for this project.

#### 3.5.3 Legacy Protocol

Older clients send a clone card with no argument. The server responds to a blank clone card by sending an "igot" card for every artifact in the repository. The client will then issue "gimme" cards to pull down all the content it needs.

The legacy protocol works well for smaller repositories (50MB with 50,000 artifacts) but is too slow and unwieldy for larger repositories. The version 2 protocol is an effort to improve performance. Further performance improvements with higher-numbered clone protocols are possible in future versions of Fossil.

### 3.6 Igot Cards

An igot card can be sent from either client to server or from server to client in order to indicate that the sender holds a copy of a particular artifact. The format is:

```
igot *artifact-id* ?*flag*?

```

The first argument of the igot card is the ID of the artifact that the sender possesses. The receiver of an igot card will typically check to see if it also holds the same artifact and if not it will request the artifact using a gimme card in either the reply or in the next message.

If the second argument exists and is "1", then the artifact identified by the first argument is private on the sender and should be ignored unless a "--private" [sync](https://fossil-scm.org/home/help?cmd=sync) is occurring.

The name "igot" comes from the English slang expression "I got" meaning "I have".

#### 3.6.1 Unversioned Igot Cards

Zero or more "uvigot" cards are sent from server to client when synchronizing unversioned content. The format of a uvigot card is as follows:

```
uvigot *name mtime hash size*

```

The *name* argument is the name of an unversioned file. The *mtime* is the last modification time of the unversioned file in seconds since 1970. The *hash* is the SHA1 or SHA3-256 hash of the unversioned file content, or "**-**" if the file has been deleted. The *size* is the uncompressed size of the file in bytes.

When the server sees a "pragma uv-hash" card for which the hash does not match, it sends uvigot cards for every unversioned file that it holds. The client will use this information to figure out which unversioned files need to be synchronized. The server might also send a uvigot card when it receives a uvgimme card but its reply message size is already oversized and hence unable to hold the usual uvfile reply.

When a client receives a "uvigot" card, it checks to see if the file needs to be transferred from client to server or from server to client. If a client-to-server transmission is needed, the client schedules that transfer to occur on a subsequent HTTP request. If a server-to-client transfer is needed, then the client sends a "uvgimme" card back to the server to request the file content.

### 3.7 Gimme Cards

A gimme card is sent from either client to server or from server to client. The gimme card asks the receiver to send a particular artifact back to the sender. The format of a gimme card is this:

```
gimme *artifact-id*

```

The argument to the gimme card is the ID of the artifact that the sender wants. The receiver will typically respond to a gimme card by sending a file card in its reply or in the next message.

The "gimme" name means "give me". The imperative "give me" is pronounced as if it were a single word "gimme" in some dialects of English (including the dialect spoken by the original author of Fossil).

#### 3.7.1 Unversioned Gimme Cards

Sync synchronizing unversioned content, the client may send "uvgimme" cards to the server. A uvgimme card requests that the server send unversioned content to the client. The format of a uvgimme card is as follows:

```
uvgimme *name*

```

The *name* is the name of the unversioned file found on the server that the client would like to have. When a server sees a uvgimme card, it normally responses with a uvfile card, though it might also send another uvigot card if the HTTP reply is already oversized.

### 3.8 Cookie Cards

A cookie card can be used by a server to record a small amount of state information on a client. The server sends a cookie to the client. The client sends the same cookie back to the server on its next request. The cookie card has a single argument which is its payload.

```
cookie *payload*

```

The client is not required to return the cookie to the server on its next request. Or the client might send a cookie from a different server on the next request. So the server must not depend on the cookie and the server must structure the cookie payload in such a way that it can tell if the cookie it sees is its own cookie or a cookie from another server. (Typically the server will embed its servercode as part of the cookie.)

### 3.9 Request-Configuration Cards

A request-configuration or "reqconfig" card is sent from client to server in order to request that the server send back "configuration" data. "Configuration" data is information about users or website appearance or other administrative details which are not part of the persistent and versioned state of the project. For example, the "name" of the project, the default Cascading Style Sheet (CSS) for the web-interface, and the project logo displayed on the web-interface are all configuration data elements.

The reqconfig card is normally sent in response to the "fossil configuration pull" command. The format is as follows:

```
reqconfig *configuration-name*

```

As of 2018-06-04, the configuration-name must be one of the following values:

|  
*   css
*   header
*   footer
*   details
*   logo-mimetype
*   logo-image
*   background-mimetype
*   background-image
*   index-page
*   timeline-block-markup
*   timeline-max-comment
*   timeline-plaintext
*   adunit
*   adunit-omit-if-admin
*   adunit-omit-if-user

 |  
*   th1-docs
*   th1-hooks
*   th1-setup
*   tcl
*   tcl-setup
*   project-name
*   short-project-name
*   project-description
*   index-page
*   manifest
*   binary-glob
*   clean-glob
*   ignore-glob
*   keep-glob
*   crlf-glob

 |  
*   crnl-glob
*   encoding-glob
*   empty-dirs
*   ~~allow-symlinks~~
*   dotfiles
*   parent-project-code
*   parent-projet-name
*   hash-policy
*   mv-rm-files
*   ticket-table
*   ticket-common
*   ticket-change
*   ticket-newpage
*   ticket-viewpage
*   ticket-editpage

 |  
*   ticket-reportlist
*   ticket-report-template
*   ticket-key-template
*   ticket-title-expr
*   ticket-closed-expr
*   xfer-common-script
*   xfer-push-script
*   xfer-commit-script
*   xfer-ticket-script
*   @reportfmt
*   @user
*   @concealed
*   @shun

 |

New configuration-names are likely to be added in future releases of Fossil. If the server receives a configuration-name that it does not understand, the entire reqconfig card is silently ignored. The reqconfig card might also be ignored if the user lacks sufficient privilege to access the requested information.

The configuration-names that begin with an alphabetic character refer to values in the "config" table of the server database. For example, the "logo-image" configuration item refers to the project logo image that is configured on the Admin page of the [web-interface](https://fossil-scm.org/home/doc/trunk/www/webui.wiki). The value of the configuration item is returned to the client using a "config" card.

If the configuration-name begins with "@", that refers to a class of values instead of a single value. The content of these configuration items is returned in a "config" card that contains pure SQL text that is intended to be evaluated by the client.

The @user and @concealed configuration items contain sensitive information and are ignored for clients without sufficient privilege.

### 3.10 Configuration Cards

A "config" card is used to send configuration information from client to server (in response to a "fossil configuration push" command) or from server to client (in response to a "fossil configuration pull" or "fossil clone" command). The format is as follows:

```
config *configuration-name size* \n *content*

```

The server will only accept a config card if the user has "Admin" privilege. A client will only accept a config card if it had sent a corresponding reqconfig card in its request.

The content of the configuration item is used to overwrite the corresponding configuration data in the receiver.

### 3.11 Pragma Cards

The client may try to influence the behavior of the server by issuing a pragma card:

```
pragma *name value...* 
```

The "pragma" card has at least one argument which is the pragma name. The pragma name defines what the pragma does. A pragma might have zero or more "value" arguments depending on the pragma name.

New pragma names may be added to the protocol from time to time in order to enhance the capabilities of Fossil. Unknown pragmas are silently ignored, for backwards compatibility.

The following are the known pragma names as of 2019-06-30:

1.  **send-private** The send-private pragma instructs the server to send all of its private artifacts to the client. The server will only obey this request if the user has the "x" or "Private" privilege.
2.  **send-catalog** The send-catalog pragma instructs the server to transmit igot cards for every known artifact. This can help the client and server to get back in synchronization after a prior protocol error. The "--verily" option to the [fossil sync](https://fossil-scm.org/home/help?cmd=sync) command causes the send-catalog pragma to be transmitted.

3.  **uv-hash** *HASH* The uv-hash pragma is sent from client to server to provoke a synchronization of unversioned content. The *HASH* is a SHA1 hash of the names, modification times, and individual hashes of all unversioned files on the client. If the unversioned content hash from the client does not match the unversioned content hash on the server, then the server will reply with either a "pragma uv-push-ok" or "pragma uv-pull-only" card followed by one "uvigot" card for each unversioned file currently held on the server. The collection of "uvigot" cards sent in response to a "uv-hash" pragma is called the "unversioned catalog". The client will used the unversioned catalog to figure out which files (if any) need to be synchronized between client and server and send appropriate "uvfile" or "uvgimme" cards on the next HTTP request.

    If a client sends a uv-hash pragma and does not receive back either a uv-pull-only or uv-push-ok pragma, that means that the content on the server exactly matches the content on the client and no further synchronization is required.

4.  **uv-pull-only** A server sends the uv-pull-only pragma to the client in response to a uv-hash pragma with a mismatched content hash argument. This pragma indicates that there are differences in unversioned content between the client and server but that content can only be transferred from server to client. The server is unwilling to accept content from the client because the client login lacks the "write-unversioned" permission.

5.  **uv-push-ok** A server sends the uv-push-ok pragma to the client in response to a uv-hash pragma with a mismatched content hash argument. This pragma indicates that there are differences in unversioned content between the client and server and that content can be transferred in either direction. The server is willing to accept content from the client because the client login has the "write-unversioned" permission.

6.  **ci-lock** *CHECKIN-HASH CLIENT-ID* A client sends the "ci-lock" pragma to the server to indicate that it is about to add a new check-in as a child of the CHECKIN-HASH check-in and on the same branch as CHECKIN-HASH. If some other client has already indicated that it was also trying to commit against CHECKIN-HASH, that indicates that a fork is about to occur, and the server will reply with a "ci-lock-fail" pragma (see below). Check-in locks automatically expire when the check-in actually occurs, or after a timeout (currently one minute but subject to change).

7.  **ci-lock-fail** *LOGIN MTIME* When a server receives two or more "ci-lock" pragma messages for the same check-in but from different clients, the second a subsequent ci-lock will provoke a ci-lock-fail pragma in the reply to let the client know that it if continues with the check-in it will likely generate a fork. The LOGIN and MTIME arguments are intended to provide information to the client to help it generate a more useful error message.

8.  **ci-unlock** *CLIENT-ID* A client sends the "ci-unlock" pragma to the server after a successful commit. This instructs the server to release any lock on any check-in previously held by that client. The ci-unlock pragma helps to avoid false-positive lock warnings that might arise if a check-in is aborted and then restarted on a branch.

Any card that begins with "#" (ASCII 0x23) is a comment card and is silently ignored.

### 3.13 Message and Error Cards

If the server discovers anything wrong with a request, it generates an error card in its reply. When the client sees the error card, it displays an error message to the user and aborts the sync operation. An error card looks like this:

```
error *error-message*

```

The error message is English text that is encoded in order to be a single token. A space (ASCII 0x20) is represented as "\s" (ASCII 0x5C, 0x73). A newline (ASCII 0x0a) is "\n" (ASCII 0x6C, x6E). A backslash (ASCII 0x5C) is represented as two backslashes "\\". Apart from space and newline, no other whitespace characters nor any unprintable characters are allowed in the error message.

The server can also send a message card that also prints a message on the client console, but which is not an error:

```
message *message-text*

```

The message-text uses the same format as an error message.

### 3.14 Unknown Cards

If either the client or the server sees a card that is not described above, then it generates an error and aborts.

## 4.0 Phantoms And Clusters

When a repository knows that an artifact exists and knows the ID of that artifact, but it does not know the artifact content, then it stores that artifact as a "phantom". A repository will typically create a phantom when it receives an igot card for an artifact that it does not hold or when it receives a file card that references a delta source that it does not hold. When a server is generating its reply or when a client is generating a new request, it will usually send gimme cards for every phantom that it holds.

A cluster is a special artifact that tells of the existence of other artifacts. Any artifact in the repository that follows the syntactic rules of a cluster is considered a cluster.

A cluster is line oriented. Each line of a cluster is a card. The cards are separated by the newline ("\n") character. Each card consists of a single character card type, a space, and a single argument. No extra whitespace and no trailing or leading whitespace is allowed. All cards in the cluster must occur in strict lexicographical order.

A cluster consists of one or more "M" cards followed by a single "Z" card. Each M card holds an argument which is an artifact ID for an artifact in the repository. The Z card has a single argument which is the lower-case hexadecimal representation of the MD5 checksum of all preceding M cards up to and included the newline character that occurred just before the Z that starts the Z card.

Any artifact that does not match the specifications of a cluster exactly is not a cluster. There must be no extra whitespace in the artifact. There must be one or more M cards. There must be a single Z card with a correct MD5 checksum. And all cards must be in strict lexicographical order.

### 4.1 The Unclustered Table

Every repository maintains a table named "**unclustered**" which records the identity of every artifact and phantom it holds that is not mentioned in a cluster. The entries in the unclustered table can be thought of as leaves on a tree of artifacts. Some of the unclustered artifacts will be other clusters. Those clusters may contain other clusters, which might contain still more clusters, and so forth. Beginning with the artifacts in the unclustered table, one can follow the chain of clusters to find every artifact in the repository.

## 5.0 Synchronization Strategies

### 5.1 Pull

A typical pull operation proceeds as shown below. Details of the actual implementation may very slightly but the gist of a pull is captured in the following steps:

1.  The client sends login and pull cards.
2.  The client sends a cookie card if it has previously received a cookie.
3.  The client sends gimme cards for every phantom that it holds.

    * * *

4.  The server checks the login password and rejects the session if the user does not have permission to pull.
5.  If the number of entries in the unclustered table on the server is greater than 100, then the server constructs a new cluster artifact to cover all those unclustered entries.
6.  The server sends file cards for every gimme card it received from the client.
7.  The server sends igot cards for every artifact in its unclustered table that is not a phantom.

    * * *

8.  The client adds the content of file cards to its repository.
9.  The client creates a phantom for every igot card in the server reply that mentions an artifact that the client does not possess.
10.  The client creates a phantom for the delta source of file cards when the delta source is an artifact that the client does not possess.

These ten steps represent a single HTTP round-trip request. The first three steps are the processing that occurs on the client to generate the request. The middle four steps are processing that occurs on the server to interpret the request and generate a reply. And the last three steps are the processing that the client does to interpret the reply.

During a pull, the client will keep sending HTTP requests until it holds all artifacts that exist on the server.

Note that the server tries to limit the size of its reply message to something reasonable (usually about 1MB) so that it might stop sending file cards as described in step (6) if the reply becomes too large.

Step (5) is the only way in which new clusters can be created. By only creating clusters on the server, we hope to minimize the amount of overlap between clusters in the common configuration where there is a single server and many clients. The same synchronization protocol will continue to work even if there are multiple servers or if servers and clients sometimes change roles. The only negative effects of these unusual arrangements is that more than the minimum number of clusters might be generated.

### 5.2 Push

A typical push operation proceeds roughly as shown below. As with a pull, the actual implementation may vary slightly.

1.  The client sends login and push cards.
2.  The client sends file cards for any artifacts that it holds that have never before been pushed - artifacts that come from local check-ins.
3.  If this is the second or later cycle in a push, then the client sends file cards for any gimme cards that the server sent in the previous cycle.
4.  The client sends igot cards for every artifact in its unclustered table that is not a phantom.

    * * *

5.  The server checks the login and push cards and issues an error if anything is amiss.
6.  The server accepts file cards from the client and adds those artifacts to its repository.
7.  The server creates phantoms for igot cards that mention artifacts it does not possess or for file cards that mention delta source artifacts that it does not possess.
8.  The server issues gimme cards for all phantoms.

    * * *

9.  The client remembers the gimme cards from the server so that it can generate file cards in reply on the next cycle.

As with a pull, the steps of a push operation repeat until the server knows all artifacts that exist on the client. Also, as with pull, the client attempts to keep the size of the request from growing too large by suppressing file cards once the size of the request reaches 1MB.

### 5.3 Sync

A sync is just a pull and a push that happen at the same time. The first three steps of a pull are combined with the first five steps of a push. Steps (4) through (7) of a pull are combined with steps (5) through (8) of a push. And steps (8) through (10) of a pull are combined with step (9) of a push.

### 5.4 Unversioned File Sync

"Unversioned files" are files held in the repository where only the most recent version of the file is kept rather than the entire change history. Unversioned files are intended to be used to store ephemeral content, such as compiled binaries of the most recent release.

Unversioned files are identified by name and timestamp (mtime). Only the most recent version of each file (the version with the largest mtime value) is retained.

Unversioned files are synchronized using the [fossil unversioned sync](https://fossil-scm.org/home/help?cmd=unversioned) command.

A schematic of an unversioned file synchronization is as follows:

1.  The client sends a "pragma uv-hash" card to the server. The argument to the uv-hash pragma is a hash of all filesnames, mtimes, and content hashes for the unversioned files held by the client.

    * * *

2.  If the unversioned content hash from the client matches the unversioned content hash on the server, then nothing needs to be done and the server no-ops. But if the hashes are different, then the server replies with either a uv-pull-only or a uv-push-ok pragma followed by uvigot cards for all unversioned files held on the server.

    * * *

3.  The client examines the uvigot cards received from the server and determines which unversioned files need to be exchanged in order to bring the client and server into synchronization. The client then sends appropriate "uvgimme" or "uvfile" cards back to the server.

    * * *

4.  The server updates its unversioned file store with received "uvfile" cards and answers "uvgimme" cards with "uvfile" cards in its reply.

The last two steps might be repeated multiple times if there is more unversioned content to be transferred than will fit comfortably in a single HTTP request.

## 6.0 Summary

Here are the key points of the synchronization protocol:

1.  The client sends one or more PUSH HTTP requests to the server. The request and reply content type is "application/x-fossil".
2.  HTTP request content is compressed using zlib.
3.  The content of request and reply consists of cards with one card per line.
4.  Card formats are:
    *   **login** *userid nonce signature*
    *   **push** *servercode projectcode*
    *   **pull** *servercode projectcode*
    *   **clone**
    *   **clone_seqno** *sequence-number*
    *   **file** *artifact-id size* **\n** *content*
    *   **file** *artifact-id delta-artifact-id size* **\n** *content*
    *   **cfile** *artifact-id size* **\n** *content*
    *   **cfile** *artifact-id delta-artifact-id size* **\n** *content*
    *   **uvfile** *name mtime hash size flags* **\n** *content*
    *   **private**
    *   **igot** *artifact-id* ?*flag*?
    *   **uvigot** *name mtime hash size*
    *   **gimme** *artifact-id*
    *   **uvgimme** *name*
    *   **cookie** *cookie-text*
    *   **reqconfig** *parameter-name*
    *   **config** *parameter-name size* **\n** *content*
    *   **pragma** *name* *value...*
    *   **error** *error-message*
    *   **message** *text-messate*
    *   **#** *arbitrary-text...*
5.  Phantoms are artifacts that a repository knows exist but does not possess.
6.  Clusters are artifacts that contain IDs of other artifacts.
7.  Clusters are created automatically on the server during a pull.
8.  Repositories keep track of all artifacts that are not named in any cluster and send igot messages for those artifacts.
9.  Repositories keep track of all the phantoms they hold and send gimme messages for those artifacts.

## 7.0 Troubleshooting And Debugging Hints

If you run the [fossil sync](https://fossil-scm.org/home/help?cmd=sync) command (or [pull](https://fossil-scm.org/home/help?cmd=pull) or [push](https://fossil-scm.org/home/help?cmd=push) or [clone](https://fossil-scm.org/home/help?cmd=clone)) with the --httptrace option, Fossil will keep a copy of each HTTP request and reply in files named:

*   `http-request-`*N*`.txt`
*   `http-reply-`*N*`.txt`

In the above, *N* is an integer that increments with each round-trip. If you are having trouble on the server side, you can run the "[fossil test-http](https://fossil-scm.org/home/help?cmd=test-http)" command in a debugger using one the "http-request-N.txt" files as input and single step through the processing performed by the server.

The "--transport-command CMD" option on [fossil sync](https://fossil-scm.org/home/help?cmd=sync) (and similar) causes the external program "CMD" to be used to move the sync message to the server and retrieve the sync reply. The CMD is given three arguments:

1.  The URL of the server
2.  The name of a temporary file that contains the output-bound sync protocol text, with the HTTP headers
3.  The name of a temporary file into which the CMD should write the reply sync protocol text, again without any HTTP headers

In a complex debugging situation, you can run the command "fossil sync --transport-command ./debugging_script" where "debugging_script" is some script of your own that invokes the anomolous behavior your are trying to debug.