<!--yml

类别：未分类

日期：2024年05月27日14:39:00

-->

# 内存保留：永久内存和Rust分配器与systemd · Graham King

> 来源：[https://darkcoding.net/software/rust-systemd-memory-remains/](https://darkcoding.net/software/rust-systemd-memory-remains/)

我们要**将三件事**连在一起，以创建在程序重新启动时**仍然存在的Rust对象**。对象背后的内存将在程序重启时继续存在。

1.  Systemd的[文件描述符存储](https://systemd.io/FILE_DESCRIPTOR_STORE/)（systemd 254+）允许我们将任何文件描述符交给systemd，然后稍后请求回来。当我们`systemctl restart <prog>`时，文件保持打开状态。是的，这类似于[套接字激活](https://darkcoding.net/software/systemd-socket-activation-in-go/)。

1.  [memfd_create](https://www.man7.org/linux/man-pages/man2/memfd_create.2.html)系统调用（Linux 3.17+）创建一个仅存在于内存中的匿名文件，只要对它的引用存在。它有效地为我们提供了指向一块分配的内存的文件描述符。它行为类似于常规文件，所以我们可以用[mmap](https://www.man7.org/linux/man-pages/man2/mmap.2.html)将其映射到常规内存中。

1.  Rust的[分配器特性](https://doc.rust-lang.org/std/alloc/trait.Allocator.html)允许我们控制对象所在的内存（至少在两年前的Rust nightly）。我们将用[Box::new_in](https://doc.rust-lang.org/std/boxed/struct.Box.html#method.new_in)来选择我们的分配器。

我们将制作一个Rust分配器，它创建一个memfd，与systemd共享，将其映射，然后使用该内存作为我们的分配器返回的分配。当程序重新启动时，我们向systemd请求我们的memfd，将其映射，并将其转换为我们的对象。最后，我们将分配器和"恢复者"包装在一个函数中，该函数会判断我们是在分配一个新对象还是恢复一个现有对象。

这是一些我们将在文中使用的任意对象：

```
// An arbitrary object we'll be using throughout this post
#[allow(dead_code)]
#[derive(Debug, Default)]
#[repr(C)]
struct Person {
    name: [u8; 8],
    credits: u32,
    age: u8,
    is_admin: bool,
    count: u32,
} 
```

注意那个相当尴尬的`name`数组。我们不想在我们的对象中有任何指针，所以不会有`String`或`Vec`，因为那些也需要存储在我们的永久内存存储中。

`repr(C)`确保了这个对象的内存布局（表示）在我们的二进制版本之间不会改变。除非你请求特定的表示，Rust不会对内存布局进行承诺。（感谢Hacker News上的Harry Stern提出这一点）。

## 一个Rust分配器

如果我们用`Box`在堆上分配一个对象，默认情况下它将使用系统分配器，在Linux上使用malloc。[观察汇编](/software/underrust-rust-assembly-output/)，我们看到它调用`__rust_alloc`（通过`GLOBAL_OFFSET_TABLE`）然后调用`__rdl_alloc`，最后调用glibc的`malloc`。

`malloc`会从操作系统获取内存（[详情](https://darkcoding.net/software/how-memory-is-allocated/)），通过移动程序断点（`sbrk`系统调用）或通过内存映射匿名区域（通过`mmap`系统调用）。

Rust（夜间版）允许我们实现我们自己的分配器。这是一个委托给系统分配器的分配器 - 如果你不知道自己使用的是哪个分配器，则是你正在使用的分配器。

```
#![feature(allocator_api)]
use std::{
    alloc::{AllocError, Allocator, GlobalAlloc, Layout, System},
    ptr::NonNull,
};

struct DelegateAlloc {}
unsafe impl Allocator for DelegateAlloc {
    fn allocate(&self, layout: Layout) -> Result<NonNull<[u8]>, AllocError> {
        unsafe {
            let ptr = System.alloc(layout);
            let slice = std::slice::from_raw_parts_mut(ptr, layout.size());
            Ok(NonNull::new_unchecked(slice))
        }
    }

    unsafe fn deallocate(&self, ptr: NonNull<u8>, layout: Layout) {
        System.dealloc(ptr.as_ptr(), layout)
    }
} 
```

我们像这样使用它：

```
 let d = DelegateAlloc {};
    let p = Box::new_in(Person::default(), d); 
```

## 由持久内存支持的分配器

现在我们知道如何编写分配器了，让我们使其使用的内存位于 systemd 拥有的文件中。我们需要调用 `memfd_create`，然后 `mmap` 该文件描述符，最后将其交给 systemd。

有两种情况：

+   创建：当我们创建新的分配时，我们需要调用 `memfd_create`，然后通过调用 [sd_pid_notify_with_fds](https://www.freedesktop.org/software/systemd/man/latest/sd_pid_notify_with_fds.html) 将该文件描述符交给 systemd。

+   恢复：当我们启动时，我们通过调用 [sd_listen_fds_with_names](https://www.freedesktop.org/software/systemd/man/latest/sd_listen_fds_with_names.html) 询问 systemd 获取 fd。然后我们只需要 `mmap`。

无论哪种情况，在 `mmap` 之后，我们都会得到一堆字节，我们需要让 Rust 相信它是一个 `Person`。这就是 `buf_addr as *mut u8 as *mut T` 做的事情（这里的 `T` 是 `Person`）。

我们使用分配器的方式如下：

```
fn main() -> anyhow::Result<()> {
    let salloc = SystemdAlloc::new("person1".to_string())?;
    let mut p1: Box<Person, SystemdAlloc> = salloc.into_box(Person::default())?;
    p1.count += 1;
    println!("{}", p1.count);
    std:🧵:sleep(std::time::Duration::from_secs(3600));
} 
```

`Person` 是一个任意的对象，在这篇文章的开头就有一个。

每个对象必须有一个名称，我们将其与 systemd 中的 FD 关联起来。在恢复时，这允许我们将正确的 FD 映射回对象。

内存仅在程序处于活动状态（即运行或重新启动）时存在。如果程序停止，systemd 将从存储中驱逐它的文件描述符。因此，在 main 结尾处有长时间的休眠。

## systemd 文件描述符存储

通常情况下，我们会创建一个 systemd 的 `.service` 文件，确保启用文件描述符存储 - 这不是默认的。在这里我们将使用 `systemd-run`：

`systemd-run --user --service-type=exec --unit=remains -p FileDescriptorStoreMax=16 ./target/debug/remains`

请注意，在这里 'remains' 是单元名称（--unit=），而不是二进制文件。然后我们将其视为任何其他 systemd 服务，只是它在用户范围内。

它的输出在 `journalctl --user -u remains` 中：

```
gktest[277943]: Received 0 fds from systemd
gktest[277943]: Creating new store
gktest[277943]: 1 
```

重新启动它：

```
systemctl --user restart remains 
```

现在 `p1.count` 增加，它在重新启动后保留：

```
systemd[4357]: Stopping remains.service
systemd[4357]: Stopped remains.service
systemd[4357]: Starting remains.service
systemd[4357]: Started remains.service
gktest[278030]: Received 1 fds from systemd
gktest[278030]: systemd sent us: person1
gktest[278030]: Restoring
gktest[278030]: 2 
```

像这样停止它：`systemctl --user stop remains`。

在那之后，`stop` 后，fd 就不再存储。如果我们现在 `start`，计数将重置。

systemd 254+（Fedora 39+）允许我们设置 `FileDescriptorStorePreserve=yes` 来保持存储在程序停止时的存储器活动。

## 改进

在过去的 [在线升级](https://darkcoding.net/software/online-upgrades-in-go/) 中，我无法重新创建的状态量非常小。我要么将该状态存储在 Redis 中，要么将其写入临时文件，作为单个序列化对象。因此，上述 systemd 分配器只能分配一次，希望这就是我们所需的全部。

要使其与多个对象一起使用，理想情况下我们可以预估我们可能需要多少空间，并在首次调用时对文件和 mmap 空间进行大小调整。如果我们想要的只是一些`String`或一个`Vec`在我们的持久化对象中，这是很有帮助的。对象可以按顺序放在文件中，我们将维护它们位置的索引。一个 mmap 可以增长（`mremap`系统调用），但不能保证有空间可以增长。

如果事先估算大小不可能，那么每当当前文件不能再增长时，我们就需要添加一个新文件。systemd 的 `FileDescriptorStoreMax` 将需要设置得足够大，并且每个文件都需要一个唯一的字符串传递给 `sd_pid_notify_with_fds` 的 `FDNAME`。一个全局索引指向文件，每个文件都有一个指向对象偏移的索引。现在我们正在编写一个真正的内存分配器。

我们可以制作一个分配器工厂（设计模式），而不是打包对象，并让每个分配器处理单个文件中的单个对象。更简单，但只适用于我们有限数量的对象。

一个有趣的练习是将分配器更改为由磁盘文件支持。磁盘上的字节将与内存中的对象布局完全相同。

## 克隆和 Pin 注释

让我们回到这个分配：

```
 let mut p1: Box<Person, SystemdAlloc> = salloc.into_box(Person::default())?; 
```

如果我们调用这个 `let p2 = p1.clone()`（在 `Person` 上添加了 `#[derive(Clone)]` 之后），`Box` 不会被克隆，而是被解引用。`clone` 直接在 `Person` 上调用。我们最终会在堆栈上得到一个新的 Person 对象。没有问题。

如果我们改用 `let p2 = Box::clone(p1)`，则 `Box` 本身会被克隆。`Box` 内部实际上是一个指针和一个分配器的元组。我们需要在 `SystemdAlloc` 上使用 `#[derive(Clone)]` 才能使其编译通过，但这只是我们问题的开始。Rust 将克隆分配器并调用其分配函数。我们的分配器只设计用于单次使用。有关此的说明，请参见前一节。

`Pin` 和一般的异步上下文并不是分配器需要担心的事情。分配中的对象可能会改变（例如，使用 `std::mem::swap`），但我们只关注分配本身，而不关心其中的内容。作为老套的程序员，我们关心的是 `Box`，而不是 `Person` :-)

开心地分配吧！

标题确实是对 Metallica / Marianne Faithfull 歌曲的引用。这并不是在 “… And Justice for All” 之后对 Metallica 的认可。
