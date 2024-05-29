<!--yml
category: 未分类
date: 2024-05-27 14:31:58
-->

# Ubuntu 22.04 Root on ZFS — OpenZFS documentation

> 来源：[https://openzfs.github.io/openzfs-docs/Getting%20Started/Ubuntu/Ubuntu%2022.04%20Root%20on%20ZFS.html](https://openzfs.github.io/openzfs-docs/Getting%20Started/Ubuntu/Ubuntu%2022.04%20Root%20on%20ZFS.html)

*   Create datasets:

    ```
    zfs  create  -o  com.ubuntu.zsys:bootfs=no  -o  canmount=off  \
      rpool/ROOT/ubuntu_$UUID/usr
    zfs  create  -o  com.ubuntu.zsys:bootfs=no  -o  canmount=off  \
      rpool/ROOT/ubuntu_$UUID/var
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/lib
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/log
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/spool

    zfs  create  -o  canmount=off  -o  mountpoint=/  \
      rpool/USERDATA
    zfs  create  -o  com.ubuntu.zsys:bootfs-datasets=rpool/ROOT/ubuntu_$UUID  \
      -o  canmount=on  -o  mountpoint=/root  \
      rpool/USERDATA/root_$UUID
    chmod  700  /mnt/root 
    ```

    The datasets below are optional, depending on your preferences and/or software choices.

    If you wish to separate these to exclude them from snapshots:

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/cache
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/lib/nfs
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/tmp
    chmod  1777  /mnt/var/tmp 
    ```

    If desired (the Ubuntu installer creates these):

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/lib/apt
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/lib/dpkg 
    ```

    If you use /srv on this system:

    ```
    zfs  create  -o  com.ubuntu.zsys:bootfs=no  \
      rpool/ROOT/ubuntu_$UUID/srv 
    ```

    If you use /usr/local on this system:

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/usr/local 
    ```

    If this system will have games installed:

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/games 
    ```

    If this system will have a GUI:

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/lib/AccountsService
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/lib/NetworkManager 
    ```

    If this system will use Docker (which manages its own datasets & snapshots):

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/lib/docker 
    ```

    If this system will store local email in /var/mail:

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/mail 
    ```

    If this system will use Snap packages:

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/snap 
    ```

    If you use /var/www on this system:

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/www 
    ```

    For a mirror or raidz topology, create a dataset for `/boot/grub`:

    ```
    zfs  create  -o  com.ubuntu.zsys:bootfs=no  bpool/grub 
    ```

    A tmpfs is recommended later, but if you want a separate dataset for `/tmp`:

    ```
    zfs  create  -o  com.ubuntu.zsys:bootfs=no  \
      rpool/ROOT/ubuntu_$UUID/tmp
    chmod  1777  /mnt/tmp 
    ```

    The primary goal of this dataset layout is to separate the OS from user data. This allows the root filesystem to be rolled back without rolling back user data.

    If you do nothing extra, `/tmp` will be stored as part of the root filesystem. Alternatively, you can create a separate dataset for `/tmp`, as shown above. This keeps the `/tmp` data out of snapshots of your root filesystem. It also allows you to set a quota on `rpool/tmp`, if you want to limit the maximum space used. Otherwise, you can use a tmpfs (RAM filesystem) later.

    **Note:** If you separate a directory required for booting (e.g. `/etc`) into its own dataset, you must add it to `ZFS_INITRD_ADDITIONAL_DATASETS` in `/etc/default/zfs`. Datasets with `canmount=off` (like `rpool/usr` above) do not matter for this.