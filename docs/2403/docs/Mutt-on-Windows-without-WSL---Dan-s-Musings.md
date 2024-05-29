<!--yml
category: 未分类
date: 2024-05-29 12:40:33
-->

# Mutt on Windows without WSL — Dan's Musings

> 来源：[https://blog.djhaskin.com/blog/mutt-on-windows-without-wsl/](https://blog.djhaskin.com/blog/mutt-on-windows-without-wsl/)

## Introduction

I have this weird hobby of trying to get good unix tools to run natively on Windows. I find this is the most efficient use of my laptop's resources, and therefore the nicest experience. WSL is nice, but it takes up a lot of RAM on my system. I prefer to use [MSYS2](https://www.msys2.org/docs/what-is-msys2/). Getting unix tools to work correctly under MSYS2 is a challenge, as we'll see, and for me, that's part of the fun.

## Prerequisites

These prerequisites are not just to use mutt, but to use msys2 successfully in any capacity for development or other work. These are the steps I follow to maximize my chances of success with msys2.

*   Msys2 needs to be installed. I like to use [scoop](https://scoop.sh/) to install it (and everything else on Windows, preferably). Just be sure to run `scoop hold msys2`. You do NOT want scoop reinstalling msys2 every half second for "updates" when you can just run `pacman -Syu` inside msys2 to get all the needed updates from there.
*   There should be no spaces in your USERPROFILE path. I know this is hard, but it's worth it. I usually do this by creating a local user named `nospaces` or something, logging in to that user, and only THEN logging into my Microsoft Windows account after my `C:\Users\<me>` folder has been created, etc.
*   Your MSYS2 installation should have this as its `/etc/fstab`, as the One True Way of ensuring your MSYS2 home directory is also your `USERPROFILE` directory:

```
# For a description of the file format, see the Users Guide
# https://cygwin.com/cygwin-ug-net/using.html#mount-table

# DO NOT REMOVE NEXT LINE. It remove cygdrive prefix from path
none / cygdrive binary,posix=0,noacl,user 0 0
C:/Users    /home   ntfs    binary,posix=0,noacl,user 0 0 
```

If you used scoop to install msys2, you can put the above file under `C:\Users\<you>\scoop\apps\msys2\current\etc\fstab` and it should work.

## Instructions

*   From a mingw64 MSYS2 prompt, type the following to install gpg, mux and msmtp:

```
pacman -S gnupg mutt mingw64/mingw-w64-x86_64-msmtp w3m 
```

*   We configure mutt as per usual, but I found that mutt can't send emails for some reason. So configure it to use [msmtp](https://marlam.de/msmtp/) for that part, and then everything works. Here is my gmail config (found under `~/.mutt/home.account`):

```
set imap_user = "djhaskin987@gmail.com"
set folder = "imaps://imap.gmail.com/"
set spoolfile="+INBOX"
set postponed = "+[Gmail]/Drafts"
set record = "+[Gmail]/Sent Mail"
set trash = "+[Gmail]/Trash"
set imap_pass = `multipass imaps://imap.gmail.com:993/`

set realname = "Daniel Jay Haskin"
set from="djhaskin987@gmail.com"
set sendmail="/mingw64/bin/msmtp -a home"

set crypt_use_gpgme
set pgp_default_key="443A163BD11CEAE798BAAB94D7268D49D06594F4"

source ~/.muttrc 
```

In this file, I use a shell script called [multipass](https://git.sr.ht/~skin/dotfiles/tree/main/item/dot-local/bin/multipass) that looks up passwords from [KeePassXC](https://keepassxc.org/) using [git-credential-keepassxc](https://github.com/Frederick888/git-credential-keepassxc).

We also see that I use `msmtp` configured to send using the home account.

Here is my `msmtp` config:

```
defaults
auth on
tls on
tls_trust_file C:/Users/bhw/scoop/apps/msys2/current/usr/ssl/certs/ca-bundle.crt
logfile ~/.msmtp.log

account home
host smtp.gmail.com
port 465
tls_starttls off
from djhaskin987@gmail.com
user djhaskin987@gmail.com
passwordeval "C:\Users\bhw\Executables\multipass.bat smtps://smtp.gmail.com:465/ djhaskin987@gmail.com"

account migadu
host smtp.migadu.com
port 465
tls_starttls off
from dan@djhaskin.com
user dan@djhaskin.com
passwordeval "C:\Users\bhw\Executables\multipass.bat smtps://smtp.migadu.com:465/ dan@djhaskin.com"

account default: home 
```

The `msmtp` program has to use a [*different* multipass](https://git.sr.ht/~skin/dotfiles/tree/main/item/Windows/Executables/multipass.bat) because it is "Windows aware" and NOT "msys2 aware", having not been compiled with [the "msys2" subsystem](https://www.msys2.org/wiki/MSYS2-introduction/), but rather with the "mingw64" subsystem. While we're on that subject, that file (above) must be written to `~/msmtprc.txt` on Windows, another gotcha I had. It's this back-and-forth windows-not-windows fence upon which msys2 teeters that makes it so important to have the msys2 home directory the same as the "windows" home directory. I find it makes things less confusing at times.

Finally, here is my `~/.muttrc`:

```
set move = no
set confirmappend = no
alternative_order text/plain text/html
auto_view text/html
set mailcap_path = ~/.mailcap

set envelope_from=yes 
```

Anyway, then I'm able to fire up and use mutt in the msys2 mingw64 terminal:

```
mutt -F ~/.mutt/home.account 
```