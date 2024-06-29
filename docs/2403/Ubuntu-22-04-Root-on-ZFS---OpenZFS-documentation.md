<!--yml

分类：未分类

日期：2024-05-27 14:31:58

-->

# Ubuntu 22.04 Root on ZFS — OpenZFS 文档

> 来源：[https://openzfs.github.io/openzfs-docs/Getting%20Started/Ubuntu/Ubuntu%2022.04%20Root%20on%20ZFS.html](https://openzfs.github.io/openzfs-docs/Getting%20Started/Ubuntu/Ubuntu%2022.04%20Root%20on%20ZFS.html)

+   创建数据集：

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

    以下数据集是可选的，具体取决于您的偏好和/或软件选择。

    如果您希望将这些分开以排除它们的快照：

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/cache
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/lib/nfs
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/tmp
    chmod  1777  /mnt/var/tmp 
    ```

    如果需要（Ubuntu 安装程序会创建这些）：

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/lib/apt
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/lib/dpkg 
    ```

    如果您在此系统上使用`/srv`：

    ```
    zfs  create  -o  com.ubuntu.zsys:bootfs=no  \
      rpool/ROOT/ubuntu_$UUID/srv 
    ```

    如果您在此系统上使用`/usr/local`：

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/usr/local 
    ```

    如果此系统将安装游戏：

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/games 
    ```

    如果此系统将有 GUI：

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/lib/AccountsService
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/lib/NetworkManager 
    ```

    如果此系统将使用 Docker（它管理自己的数据集和快照）：

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/lib/docker 
    ```

    如果此系统将在`/var/mail`存储本地电子邮件：

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/mail 
    ```

    如果此系统将使用 Snap 包：

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/snap 
    ```

    如果您在此系统上使用`/var/www`：

    ```
    zfs  create  rpool/ROOT/ubuntu_$UUID/var/www 
    ```

    对于镜像或raidz拓扑，创建一个数据集用于`/boot/grub`：

    ```
    zfs  create  -o  com.ubuntu.zsys:bootfs=no  bpool/grub 
    ```

    推荐稍后使用 tmpfs，但如果您想为`/tmp`创建一个单独的数据集：

    ```
    zfs  create  -o  com.ubuntu.zsys:bootfs=no  \
      rpool/ROOT/ubuntu_$UUID/tmp
    chmod  1777  /mnt/tmp 
    ```

    此数据集布局的主要目标是将操作系统与用户数据分开。这允许在不回滚用户数据的情况下回滚根文件系统。

    如果您什么都不做，`/tmp`将作为根文件系统的一部分存储。或者，您可以像上面那样为`/tmp`创建一个单独的数据集。这样可以将`/tmp`数据排除在根文件系统的快照之外。如果您希望限制最大使用空间，还可以在`rpool/tmp`上设置配额。否则，您可以稍后使用 tmpfs（RAM 文件系统）。

    **注意：** 如果将启动所需的目录（例如`/etc`）分开到自己的数据集中，您必须将其添加到`/etc/default/zfs`中的`ZFS_INITRD_ADDITIONAL_DATASETS`。具有`canmount=off`（如上面的`rpool/usr`）的数据集对这没有影响。
