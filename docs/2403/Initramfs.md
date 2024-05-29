<!--yml
category: 未分类
date: 2024-05-27 14:44:16
-->

# Initramfs

> 来源：[https://blog.izissise.net/posts/initramfs/](https://blog.izissise.net/posts/initramfs/)

With a compiled kernel image, it can be useful to load your own [initramfs](https://en.wikipedia.org/wiki/Initial_ramdisk):

*   make it as small as possible considering the features you need
*   create a minimal test environment for the kernel image
*   mount a remote filesystem as the root
*   your own reason

Anyways, with the [official documentation](https://www.kernel.org/doc/Documentation/filesystems/ramfs-rootfs-initramfs.txt) help, let's create an initramfs.

We need to gen_init_cpio tool that can found in the kernel repository:

`gen_init_cpio.c` needs to be compiled:

```
wget -O gen_init_cpio.c 'https://raw.githubusercontent.com/torvalds/linux/v6.7/usr/gen_init_cpio.c' gcc -o gen_init_cpio gen_init_cpio.c 
```

Now it is ready for use.

Let's create an hello world init program that the kernel can execute to show that everything works correctly:

```
printf '%s\n' '#include <stdio.h>' '#include <unistd.h>' 'int main() { printf("Hello world!\n"); sleep(999999999); }' | gcc -fpic -static -xc - -o init 
```

Here is a basic initramfs description, we give it to `gen_init_cpio` directly:

```
./gen_init_cpio -t 0 -c - > initramfs.cpio <<EOF # this is a file containing newline separated entries that # describe the files to be included in the initramfs archive: # # a comment # file <name> <location> <mode> <uid> <gid> [<hard links>] # dir <name> <mode> <uid> <gid> # nod <name> <mode> <uid> <gid> <dev_type> <maj> <min> # slink <name> <target> <mode> <uid> <gid> # pipe <name> <mode> <uid> <gid> # sock <name> <mode> <uid> <gid>   # Root directory dir . 1755 0 0   # common directories dir proc 1775 0 0 dir dev 1775 0 0 dir bin 1775 0 0 dir sys 1775 0 0 dir run 1775 0 0 dir mnt 1775 0 0 dir etc 1775 0 0   # symlink /sbin -> /bin slink bin /sbin 1744 0 0   # common devices dir dev/pts 1775 0 0 dir dev/shm 1775 0 0 dir dev/udev 1775 0 0 nod dev/zero 1666 0 0 c 1 5 nod dev/null 1666 0 0 c 1 3 nod dev/console 1622 0 0 c 5 1 nod dev/loop0 1644 0 0 b 7 0 nod dev/ttyS0 1666 0 0 c 4 64 nod dev/tty 1666 0 0 c 5 0 nod dev/ptmx 1666 0 0 c 5 2 nod dev/random 1444 0 0 c 1 8 nod dev/urandom 1444 0 0 c 1 9 slink dev/core /proc/kcore 1777 0 0 slink dev/fd /proc/self/fd 1777 0 0 slink dev/stdout /proc/self/fd/1 1777 0 0 slink dev/stdin /proc/self/fd/0 1777 0 0 slink dev/stderr /proc/self/fd/2 1777 0 0   # include previously compiled init # make sure the path is /init file /init init 0755 0 0 EOF 
```

The line starting with `file` takes an external file as a second parameter, and that's how we include the `init` program in the archive.

Note this is rootless, we don't need to re-create the filesystem tree locally or use fakeroot to create special file in the archive.

If you're following along, you now have created `initramfs.cpio` archive!

With our initramfs we just need a linux kernel to be able to run it in qemu:

```
qemu-system-${HOSTTYPE} -kernel vmlinuz-6.7.5-060705-generic -initrd initramfs.cpio -append "console=ttyS0" -cpu max -m 128M -nographic -serial mon:stdio -nodefaults 
```

You can find pre-compiled kernel image in [ubuntu repository](https://kernel.ubuntu.com/mainline/?C=M;O=D)

Happy hacking!