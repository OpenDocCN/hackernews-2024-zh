<!--yml
category: 未分类
date: 2024-05-29 13:23:17
-->

# Editing remote files via scp in vim | Vim Tips Wiki | Fandom

> 来源：[https://vim.fandom.com/wiki/Editing_remote_files_via_scp_in_vim](https://vim.fandom.com/wiki/Editing_remote_files_via_scp_in_vim)

**Duplicate tip**

This tip is very similar to the following:

These tips need to be merged – see the [merge guidelines](/wiki/Vim_Tips_Wiki:Merge_guidelines "Vim Tips Wiki:Merge guidelines").

Vim 6.x has the netrw plugin installed as a standard plugin. It allows you to edit files via ftp, rcp, scp, or http. If your username differs on the remote host, however, and you're trying to use scp, things can get a little weird, particularly if you're not editing a document under your user tree.

To get around this, try opening the file as follows:

```
vim scp://remoteuser@server.tld//absolute/path/to/document

```

Similarly, you can open the file from within Vim in a new buffer by running:

```
:e scp://remoteuser@server.tld//absolute/path/to/document

```

or in a new tab using:

```
:tabe scp://remoteuser@server.tld//absolute/path/to/document

```

Notice two things:

1.  `remoteuser@`: This is used to specify the user name on the remote server. Without this, it will use the user's name on the local computer. Often that will come from the $USERNAME environment variable. If the user has the same name on the local computer and the remote server, this part is unnecessary. If you're unsure whether it's needed, use it just to be safe.
2.  Double slashes ("`//`") between the hostname and file path: At least one slash is needed to separate the remote server's hostname from the file path. That slash is not included in the path used to reference the file on the remote server. If the path to the file is absolute, then it must begin with a slash, giving two slashes between the hostname and file path as shown above.

    However, if the file to be edited is contained within the home directory of the remote user, a relative path may be used, which should not use a second slash. For example, if the absolute path to the file to be edited is `/users/remoteuser/relative/path/to/document` and the home directory for remoteuser is `/users/remoteuser`, then the following command will open that file:

    ```
    vim scp://remoteuser@server.tld/relative/path/to/document
    ```

## [[](/wiki/Editing_remote_files_via_scp_in_vim?veaction=edit&section=1 "Edit section: Comments")]

The [latest netrw.vim](http://www.drchip.org/astronaut/vim/index.html#NETRW) has several improvements. Later Windows ftp is handled, new protocols (rsync, cadaver, fetch), user fixup functions, etc.

* * *

How can we store the password? It prompts for password each time we save!

* * *

I just got this working on Win2k w/ PuTTY's command line scp program [http://www.chiark.greenend.org.uk/~sgtatham/putty/](http://www.chiark.greenend.org.uk/~sgtatham/putty/)

*   copy pscp.exe into your path somewhere as scp.exe
*   put "let g:netrw_cygwin= 0" in your $VIM/_vimrc

* * *

A possible gotcha:

If you don't put the path as specified (and noted) in the tip, you may get a non-intuitive error: not putting "//" between the hostname and the *absolute* path of the file you edit may cause vim to try to retrieve the file via rcp, as in

```
:!rcp scp://m@mymachine.com:t1

```

and result in an error. Also be careful that you put the absolute path of the file on the remote machine, not the path relative to the remote user's home directory.

* * *

Using relative paths is quite normal and well supported. Try

```
:r scp://m@machine/t1

```

* * *

Someone was asking if you could define the port for ftp

```
vim ftp://[user@]machine[[:#]portnumber]/path

```

try that...just like any other url.

```
vim ftp://stankonia@domainname.com:6090/public_html/index.html

```

I guess that would work.

* * *

There is a nice way to save your passwords: Create .netrc under you home directory and put lines in, one per ftp machine, like this one:

```
machine yourftp.somewhere.org login yourlogin password "yoursecret"

```

* * *

ftp remote edit is OK.

Run command:

```
gvim <host>//<path_2_file> ftp://<host>//<path_2_file>;

```

Then enter user name and password.

* * *

If everything seems to be setup correctly but you're still unable to access a file with ftp. Check the permissions on your .netrc file. If .netrc is readable by anyone else besides the owner then ftp auto fails.

```
chmod 600 .netrc

```

* * *

Solved for me on windows with putty, in vimrc:

```
let g:netrw_cygwin = 0
let g:netrw_scp_cmd = "\"C:\\Program Files\\PuTTY\\pscp.exe\" -pw mypasswd "

```

and now run from wincommander:

```
gvim scp://user@server.xxx.cz/file.txt

```

* * *

Get the latest version to fix any problems you are having.

* * *

Was getting a bit annoyed with having to type the full path a remote user's file when I'm using scp and connecting as root on the remote end to edit a local user's file on the remote site and found out that I could do this and vim did "The Right Thing"

```
vim scp://root@example.com/~user/public_html/.htaccess

```

That was a lot nicer than having to bother with:

```
vim scp://root@example.com//home/user/public_html/.htaccess

```

Maybe not such a pain in that example, but if you're working with an Ensim for Linux system, you've got everything chrooted which makes you have to type a ridiculously long path such as:

```
vim scp://root@example.com//home/virtual/site2/fst/var/www/html/.htaccess (yawn)

```

* * *

To change the scp port, there's several options. A quick one would be while you've opened vim to type this:

```
:let g:netrw_scp_cmd="scp -q -P <desired_new_port>"

```

and then just type:

```
:e scp://my_user@remote_hostname//path/to/remote/file

```

--> I think a better solution is to use ssh-mechanisms, i.e. the ~/.ssh/config file:

```
Host lala
 HostName test.machine.example.net
 User remoteuser
 IdentityFile ~/.ssh/id_for_test.machine
 Port 57

```

* * *

I have discovered how to make passive mode ftp work. See [http://alecthegeek.github.io/blog/2007/02/06/handy-hack-how-to-use-vim-netrw-in-ftp-passive-mode/](http://alecthegeek.github.io/blog/2007/02/06/handy-hack-how-to-use-vim-netrw-in-ftp-passive-mode/)

* * *

Try "C:\Program Files\Vim\vim71\gvim.exe" --remote-tab !.! in WinSCP to open each file in a separate tab in the same gVim instance. I also clicked the "External Editor Opens Multiple Files in one window" checkbox. There is another option --remote-silent that will suppress the first warning that there is no gVim already running, but you cannot use it with the --remote-tab option. I prefer to ack. the one warning rather than making sure I'm in a new tab before opening anything.----

--Preceding [unsigned](https://en.wikipedia.org/wiki/Signatures "wikipedia:Signatures") comment added by [161.88.255.139](/wiki/User:161.88.255.139 "User:161.88.255.139") (talk • [contribs](/wiki/Special:Contributions/161.88.255.139 "Special:Contributions/161.88.255.139")) 20:49, 18 April 2008

* * *

Use "C:\Program Files\Vim\vim71\gvim.exe" --remote-tab-silent !..! in WinSCP as External Editor works just fine.

* * *

For windows I would suggest adding PuTTY to the system path (or not) and simply set the g:netrw_cygwin=0, and g:netrw_scp_cmd=<PATH TO PSCP> and use Pageant to manage your private keys; that way you don't have to keep your password in your vimrc file. Pageant provides the key to PSCP when it tries to connect.

--Preceding [unsigned](https://en.wikipedia.org/wiki/Signatures "wikipedia:Signatures") comment added by [216.145.54.7](/wiki/User:216.145.54.7 "User:216.145.54.7") (talk • [contribs](/wiki/Special:Contributions/216.145.54.7 "Special:Contributions/216.145.54.7")) 11:56 UTC, 20 May 2009

* * *

[bcvi](http://sshmenu.sourceforge.net/articles/bcvi/) is a utility that works with SSH/SCP and the Vim NetRW plugin described above. When you log into a remote server with SSH+bcvi, you can `cd` into any directory then type `vi filename` and the gvim command will get launched back on your workstation, with the correct SCP URL to point to the file on the server. If that sounds confusing, the [bcvi article](http://sshmenu.sourceforge.net/articles/bcvi/) clarifies things with examples and pictures.

* * *