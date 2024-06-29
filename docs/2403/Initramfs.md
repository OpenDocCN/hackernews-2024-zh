<!--yml

category: 未分类

date: 2024-05-27 14:44:16

-->

# Initramfs

> 来源：[https://blog.izissise.net/posts/initramfs/](https://blog.izissise.net/posts/initramfs/)

有了编译过的内核镜像，加载您自己的[initramfs](https://en.wikipedia.org/wiki/Initial_ramdisk)可能会很有用：

+   使它在满足所需功能的同时尽可能小

+   为内核镜像创建一个最小的测试环境。

+   将远程文件系统挂载为根文件系统

+   您自己的理由

总之，借助[官方文档](https://www.kernel.org/doc/Documentation/filesystems/ramfs-rootfs-initramfs.txt)的帮助，让我们创建一个initramfs。

我们需要的gen_init_cpio工具可以在内核仓库中找到：

`gen_init_cpio.c` 需要编译：

```
wget -O gen_init_cpio.c 'https://raw.githubusercontent.com/torvalds/linux/v6.7/usr/gen_init_cpio.c' gcc -o gen_init_cpio gen_init_cpio.c 
```

现在可以使用了。

让我们创建一个hello world的init程序，让内核能够执行并显示一切正常：

```
printf '%s\n' '#include <stdio.h>' '#include <unistd.h>' 'int main() { printf("Hello world!\n"); sleep(999999999); }' | gcc -fpic -static -xc - -o init 
```

这是一个基本的initramfs描述，我们直接交给`gen_init_cpio`：

```
./gen_init_cpio -t 0 -c - > initramfs.cpio <<EOF # this is a file containing newline separated entries that # describe the files to be included in the initramfs archive: # # a comment # file <name> <location> <mode> <uid> <gid> [<hard links>] # dir <name> <mode> <uid> <gid> # nod <name> <mode> <uid> <gid> <dev_type> <maj> <min> # slink <name> <target> <mode> <uid> <gid> # pipe <name> <mode> <uid> <gid> # sock <name> <mode> <uid> <gid>   # Root directory dir . 1755 0 0   # common directories dir proc 1775 0 0 dir dev 1775 0 0 dir bin 1775 0 0 dir sys 1775 0 0 dir run 1775 0 0 dir mnt 1775 0 0 dir etc 1775 0 0   # symlink /sbin -> /bin slink bin /sbin 1744 0 0   # common devices dir dev/pts 1775 0 0 dir dev/shm 1775 0 0 dir dev/udev 1775 0 0 nod dev/zero 1666 0 0 c 1 5 nod dev/null 1666 0 0 c 1 3 nod dev/console 1622 0 0 c 5 1 nod dev/loop0 1644 0 0 b 7 0 nod dev/ttyS0 1666 0 0 c 4 64 nod dev/tty 1666 0 0 c 5 0 nod dev/ptmx 1666 0 0 c 5 2 nod dev/random 1444 0 0 c 1 8 nod dev/urandom 1444 0 0 c 1 9 slink dev/core /proc/kcore 1777 0 0 slink dev/fd /proc/self/fd 1777 0 0 slink dev/stdout /proc/self/fd/1 1777 0 0 slink dev/stdin /proc/self/fd/0 1777 0 0 slink dev/stderr /proc/self/fd/2 1777 0 0   # include previously compiled init # make sure the path is /init file /init init 0755 0 0 EOF 
```

以`file`开头的行以第二个参数作为外部文件，并且这是我们在存档中包含`init`程序的方式。

请注意这是无根的，我们无需在本地重新创建文件系统树或使用fakeroot在存档中创建特殊文件。

如果您跟随操作，您现在已经创建了`initramfs.cpio`存档！

使用我们的initramfs，我们只需一个Linux内核就可以在qemu中运行它：

```
qemu-system-${HOSTTYPE} -kernel vmlinuz-6.7.5-060705-generic -initrd initramfs.cpio -append "console=ttyS0" -cpu max -m 128M -nographic -serial mon:stdio -nodefaults 
```

您可以在[Ubuntu仓库](https://kernel.ubuntu.com/mainline/?C=M;O=D)找到预编译的内核镜像。

祝你编程愉快！
