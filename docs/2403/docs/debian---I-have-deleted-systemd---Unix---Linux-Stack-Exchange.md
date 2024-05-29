<!--yml
category: 未分类
date: 2024-05-29 12:38:11
-->

# debian - I have deleted systemd - Unix & Linux Stack Exchange

> 来源：[https://unix.stackexchange.com/questions/772960/i-have-deleted-systemd](https://unix.stackexchange.com/questions/772960/i-have-deleted-systemd)

I'll assume all commands below are run as root user. Either login directly as root, or if you can't, log instead as the user allowed to use `sudo` and use `sudo -i`: there should be a root prompt with a `#` at its end.

You should backup the contents of `/var/log/apt` with

```
cp -a /var/log/apt /root/log-apt-backup 
```

so you can later find what other deleted package you might have to install back.

As support for system V is slowly degrading, I can assume that some GUI parts rely on systemd features and thus fail to start.

You should reinstall `systemd`, but this might not be enough: to trigger additional needed dependencies, instead you should remove what provides System V's `/sbin/init`: `sysvinit-core`. The same way removing `systemd` triggered installation of replacement system V packages, removing this one will trigger replacement in the systemd ecosystem.

Tested this on an empty LXC container:

```
root@bookworm-test:~# apt remove systemd
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  initscripts insserv orphan-sysvinit-scripts sensible-utils startpar sysv-rc
  sysvinit-core ucf
Suggested packages:
  bootchart2 bootlogd
The following packages will be REMOVED:
  dbus-user-session libnss-systemd libpam-systemd systemd systemd-resolved
  systemd-sysv
The following NEW packages will be installed:
  initscripts insserv orphan-sysvinit-scripts sensible-utils startpar sysv-rc
  sysvinit-core ucf
0 upgraded, 8 newly installed, 6 to remove and 0 not upgraded. 
```

and later after reboot fixes etc (don't run this, won't be enough):

```
# apt install systemd
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  systemd-timesyncd
Suggested packages:
  systemd-container systemd-homed systemd-userdbd systemd-boot
  systemd-resolved libqrencode4 libtss2-esys-3.0.2-0 libtss2-mu0 libtss2-rc0
  polkitd | policykit-1
The following NEW packages will be installed:
  systemd systemd-timesyncd
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded. 
```

won't do enough to be useful (it would stay with sys V init).

Instead:

```
root@bookworm-test:~# apt remove sysvinit-core
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  initscripts insserv startpar sysv-rc
Use 'apt autoremove' to remove them.
The following additional packages will be installed:
  dbus-user-session libnss-systemd libpam-systemd systemd systemd-sysv
  systemd-timesyncd
Suggested packages:
  systemd-container systemd-homed systemd-userdbd systemd-boot
  systemd-resolved libqrencode4 libtss2-esys-3.0.2-0 libtss2-mu0 libtss2-rc0
  polkitd | policykit-1
The following packages will be REMOVED:
  sysvinit-core
The following NEW packages will be installed:
  dbus-user-session libnss-systemd libpam-systemd systemd systemd-sysv
  systemd-timesyncd
0 upgraded, 6 newly installed, 1 to remove and 0 not upgraded. 
```

appears more useful. At least one suggested package should be installed back: `polkitd` (by appending a `+` at the end, this can be done within a `remove` operation). So in the end run:

```
apt remove sysvinit-core polkitd+ 
```

This requires network. It appears OP (and actually my LXC test) doesn't have network anymore by default. This should be fixable easily with a wired Ethernet interface named `eth0` (actual name can be retrieved from `ip -br link`'s results, replace `eth0` below with it if needed):

```
dhclient -v eth0 
```

then try again the previous `apt` command.

on a minimal system something like this would be displayed:

```
# apt remove sysvinit-core polkitd+
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  initscripts insserv startpar sysv-rc
Use 'apt autoremove' to remove them.
The following additional packages will be installed:
  dbus-user-session libduktape207 libglib2.0-0 libglib2.0-data libicu72
  libnss-systemd libpam-systemd libpolkit-agent-1-0 libpolkit-gobject-1-0
  libxml2 sgml-base shared-mime-info systemd systemd-sysv systemd-timesyncd
  xdg-user-dirs xml-core
Suggested packages:
  low-memory-monitor polkitd-pkla sgml-base-doc systemd-container
  systemd-homed systemd-userdbd systemd-boot systemd-resolved libqrencode4
  libtss2-esys-3.0.2-0 libtss2-mu0 libtss2-rc0 debhelper
The following packages will be REMOVED:
  sysvinit-core
The following NEW packages will be installed:
  dbus-user-session libduktape207 libglib2.0-0 libglib2.0-data libicu72
  libnss-systemd libpam-systemd libpolkit-agent-1-0 libpolkit-gobject-1-0
  libxml2 polkitd sgml-base shared-mime-info systemd systemd-sysv
  systemd-timesyncd xdg-user-dirs xml-core
0 upgraded, 18 newly installed, 1 to remove and 0 not upgraded. 
```

and reboot, to switch back to systemd. This might or might not restore the GUI, but this should have switched back to `systemd`. To verify, one can run (as root):

```
ldd /proc/1/exe | grep systemd 
```

which should have non-empty output.

Once above is done, examine the log backup in `/root/log-apt-backup/history.log` and search for a line looking like the commands you did: `Commandline: apt remove systemd` and `Commandline: apt autoremove systemd`. Check all removed packages shown below their respective `Remove:` entry, and add back what might still be missing.