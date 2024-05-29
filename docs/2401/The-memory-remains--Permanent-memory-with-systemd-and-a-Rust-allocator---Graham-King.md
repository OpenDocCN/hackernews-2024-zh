<!--yml
category: 未分类
date: 2024-05-27 14:39:00
-->

# The memory remains: Permanent memory with systemd and a Rust allocator · Graham King

> 来源：[https://darkcoding.net/software/rust-systemd-memory-remains/](https://darkcoding.net/software/rust-systemd-memory-remains/)

We are going to **stitch three things** together to make Rust objects that **survive program restart**. The memory backing the object will continue existing while the program restarts.

1.  Systemd’s [File Descriptor Store](https://systemd.io/FILE_DESCRIPTOR_STORE/) (systemd 254+) allows us to give any file descriptor to systemd, and later ask for it back. The file stays open when we `systemctl restart <prog>`. Yes this is similar to [socket activation](https://darkcoding.net/software/systemd-socket-activation-in-go/).

2.  The [memfd_create](https://www.man7.org/linux/man-pages/man2/memfd_create.2.html) syscall (Linux 3.17+) creates an anonymous file that only exists in memory, and only as long as a reference to it exists. It effectively gives us a file descriptor pointing to a chunk of allocated memory. It behaves like a regular file so we can map it into regular memory with [mmap](https://www.man7.org/linux/man-pages/man2/mmap.2.html).

3.  Rust’s [Allocator trait](https://doc.rust-lang.org/std/alloc/trait.Allocator.html) which allows us to control the memory an object lives in (Rust nightly since at least 2 years ago). We will select our allocator with [Box::new_in](https://doc.rust-lang.org/std/boxed/struct.Box.html#method.new_in).

We will make a Rust allocator which creates a memfd, shares it with systemd, mmap’s it, and uses that memory as the allocation our allocator returns. When the program restarts we ask systemd for our memfds, mmap it, and convert that into our object. Finally we wrap the allocator and the ‘restorer’ in a function which figures out whether we are allocating a new object or recovering an existing one.

Here’s an arbitrary object we will use throught the post:

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

Note the rather awkward `name` array. We don’t want any pointers in our object, so no `String` or `Vec` because those would also need to go in our permanent memory store.

The `repr(C)` ensures that the in-memory layout (representation) of this object doesn’t change between versions of our binary. Rust makes no promises on memory layout unless you request a specific representation. (Thank you Harry Stern on Hacker News for catching this).

## A Rust Allocator

If we allocate an object on the heap with `Box`, by default it will use the sytem allocator, which on Linux uses malloc. [Looking at the assembly](/software/underrust-rust-assembly-output/) we see it call `__rust_alloc` (via `GLOBAL_OFFSET_TABLE`) which calls `__rdl_alloc` which calls glibc’s `malloc`.

`malloc` will get memory from the operating system ([details](https://darkcoding.net/software/how-memory-is-allocated/)) by either moving the program break (`sbrk` syscall) or by memory mapping an anonymous region (via `mmap` syscall).

Rust (nightly) allows us to implement our own allocator. Here is an allocator that delegates to the system allocator - the allocator you’re using if you don’t know which one you’re using.

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

We use it like this:

```
 let d = DelegateAlloc {};
    let p = Box::new_in(Person::default(), d); 
```

## An allocator backed by persistent memory

Now that we know how to write an allocator, let’s make the memory it uses be in a file systemd owns. We need to call `memfd_create`, then `mmap` that file descriptor, and finally give it to systemd.

There are two cases:

*   Create: When we are creating a new allocation we need to call `memfd_create`, and then give that file descriptor to systemd by calling [sd_pid_notify_with_fds](https://www.freedesktop.org/software/systemd/man/latest/sd_pid_notify_with_fds.html).

*   Restore: When we are starting up we ask systemd for the fd by calling [sd_listen_fds_with_names](https://www.freedesktop.org/software/systemd/man/latest/sd_listen_fds_with_names.html). Then we only need to `mmap`.

In either case after `mmap` we end up with a bag of bytes, which we need to convince Rust is a `Person`. That’s what `buf_addr as *mut u8 as *mut T` is doing (here `T` is `Person`).

We use the allocator like this:

```
fn main() -> anyhow::Result<()> {
    let salloc = SystemdAlloc::new("person1".to_string())?;
    let mut p1: Box<Person, SystemdAlloc> = salloc.into_box(Person::default())?;
    p1.count += 1;
    println!("{}", p1.count);
    std:🧵:sleep(std::time::Duration::from_secs(3600));
} 
```

`Person` is an arbitrary object, there’s one at the start of this post.

Each object must have a name which we associate with the FD in systemd. On restore that allows us to map the correct FD back to the object.

The memory only exists while the program is active (meaning running or restarting). If the program is stopped systemd will evict it’s file descriptor from the store. Hence the long sleep at the end of main.

## systemd file descriptor store

Typically now we’d create a systemd `.service` file making sure to enable the File Descriptor store - it doesn’t default. Here we’ll use `systemd-run`:

`systemd-run --user --service-type=exec --unit=remains -p FileDescriptorStoreMax=16 ./target/debug/remains`

Note that here ‘remains’ is the unit name (–unit=) not the binary. Then we treat it like any other systemd service except it’s in the user scope.

It’s output is in `journalctl --user -u remains`:

```
gktest[277943]: Received 0 fds from systemd
gktest[277943]: Creating new store
gktest[277943]: 1 
```

Restart it:

```
systemctl --user restart remains 
```

Now `p1.count` increases, it survives restart:

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

Stop it like this: `systemctl --user stop remains`.

After that `stop` the fd is no longer stored. If we `start` now the count will reset.

systemd 254+ (Fedora 39+) allows us to set `FileDescriptorStorePreserve=yes` to keep the store alive even when our program is stopped.

## Improvements

When I have done [online upgrades in the past](https://darkcoding.net/software/online-upgrades-in-go/) the amount of state that I couldn’t recreate was quite small. I either stored that state in Redis or wrote it to a temporary file, as a single serialized object. Hence the systemd allocator above can only allocate once, hopefully that’s all we’d need.

To make it work with multiple objects ideally we could estimate up-front how much space we might need, and size the file and mmap space on first call. This is helpful if all we want is a couple of `String` or a `Vec` in we persistent object. The objects could go sequentially in the file, we’d maintain an index of their positions. An mmap can grow (`mremap` syscall) but there’s no guarantee that there is space to grow into.

If estimating size up-front is not possible we’d need to add a new file whenever the current one can no longer grow. systemd’s `FileDescriptorStoreMax` will need to be set large enough, and each file will need a unique string passed to `sd_pid_notify_with_fds`’s `FDNAME`. A global index points to the file, and each file has an index of offsets to the ojects. And now we’re writing a real memory allocator.

Instead of packing the objects we could make an allocator factory (the design pattern) and have each allocator deal with a single object in a single file. Simpler, but only suitable if we have a limited number of objects.

A fun exercise is to change the allocator to be backed by a disk file. The bytes on disk will be exactly how the object is laid out in memory.

## Clone and Pin notes

Let’s go back to this allocation:

```
 let mut p1: Box<Person, SystemdAlloc> = salloc.into_box(Person::default())?; 
```

If we call this `let p2 = p1.clone()` (after adding `#[derive(Clone)]` to `Person`) the Box isn’t cloned but dereferenced. `clone` is called directly on `Person`. We end up with a new Person object on the stack. No problems there.

If instead we do `let p2 = Box::clone(p1)` the box itself is cloned. A `Box` internally is a tuple of a pointer and an allocator. We would need to `#[derive(Clone)]` on `SystemdAlloc` for it to compile, but that’s just where our problems will start. Rust will clone the allocator and call allocate on it. Our allocator is only designed for single use. See the previous section for notes on that.

`Pin` and async contexts in general are not something an allocator needs to worry about. The object in the allocation might change (e.g. with `std::mem::swap`) but we are only concerned with the allocation itself, not with what’s inside it. As cliched programmers, we care about the `Box`, not the `Person` :-)

Happy allocating!

The title is indeed a reference to the Metallica / Marianne Faithfull song. This is not an endorsement of Metallica after “… And Justice for All”.