<!--yml
category: 未分类
date: 2024-05-27 12:54:12
-->

# Containers and Unikernels: Conceptually Similar, Fundamentally Different, and Inextricably Intertwined. — Unikraft

> 来源：[https://unikraft.io/blog/containers-and-unikernels/](https://unikraft.io/blog/containers-and-unikernels/)

The paradox of approaching an article which compares and contrasts unikernels and containers, I think, is that at Unikraft, we’ve embraced containers as one of our own. We are of course a unikernel company, which sounds like it doesn’t make any sense. Why would a unikernel company, who develop tools to build and run lightweight VMs as an alternative to containers, advocate for the use of containers? Are you mad, this must be blasphemy!

It’s really just a case that the landscape for containers and unikernels has evolved considerably since the first musings of containers and unikernels were muttered together in the same sentence. In fact, ultimately, I think it’s no longer correct to say that they are two separate ideologies for approaching units of computation. This is because both “containers” and “unikernels” have been significantly developed over the last 5-10 years. These two terms pack so much more punch than they used to and there are very good characterizable properties of both technology areas which in fact work very well together. Let’s explore.

## From Chaos to Cargo Cult

Whether it was Docker itself or another tool in its shoes, the arrival of the container scene in the early 2010s stemmed from several challenges developers faced in the pre-container era:

*   **Inconsistent Environments**: Applications often behaved differently depending on the underlying operating system and its configuration. This made development, testing, and deployment a frustrating process.

*   **Virtual Machines (VMs) Were Bulky**: VMs, the dominant technology for application isolation at the time, were resource-intensive and slow to boot. This hampered development workflows and deployment agility.

*   **Dependency Management Headaches**: Managing application dependencies across different environments was a complex and error-prone task. Developers needed a way to ensure their applications had everything they needed to run correctly.

Docker’s journey began in 2008, where, in fact, it was not alone in tackling these problems. At the time, it was already piggy-backing on much existing ground-work of base technologies, including: of course everybody’s favorite `chroot`, the lesser-known [Capsicum](https://www.usenix.org/legacy/event/sec10/tech/full_papers/Watson.pdf), and the rise of LXC (Linux Containers), to name a few. LXC provided a way to isolate processes on a Linux system and was one of the first major steps towards the container-based solutions we have today, but it lacked a user-friendly ecosystem and was, per its name, Linux-only.

Enter Docker in 2013.

Built on top of LXC, Docker introduced a complete container management system and, arguably, one of the most important contributions: *the container image*.

## `FROM` Blob with Love

Ultimately, Docker’s initial mission was to enable consistency across an increasingly heterogeneous landscape of execution environments — to be gone with the ways of “it works on my machine” — and to make approaching this more straightforward than how it existed at the time. Appropriately, Docker is praised for its tooling and we and many others embrace this mantra in our own projects: to make a hard problem easier to tackle with tooling, and especially tooling that is straight forward to use, well documented and stable. Such is the echoes of UNIX philosophy.

Beyond its immediate CLI, moreover, there were three key innovations which were significantly developed by Docker and then by many others in the container community since which we embrace here at Unikraft: the definition, storage and transportation of file collections:

1.  The `Dockerfile` is a remarkably simple and extremely elegant approach to addressing the inconsistent environments problem. By using a forcefully basic syntax and to allow for cross-referencing between existing filesystems (`FROM` and `COPY`/`ADD`), the blueprint for defining a filesystem became, not only easy, but extensible.

2.  (and 3.) By introducing rich metadata for the resulting filesystem itself, “the image” (a collection of JSON files describing things, including other files or blobs), as well as each transaction of change (overlay) to the filesystem, “the layers”; the overall composition of the resulting filesystem became inherently more practical. It enabled interesting distribution and caching mechanisms as well as more informed look ups.

Whilst unikernels were born out of the security domain and containers out of reproducibility (achieved through isolation), today, containers and unikernels still share a core principle: software encapsulation. Both create isolated environments for applications. Containers achieve this by sharing the host kernel but isolating the application’s processes, libraries, and files. Unikernels, on the other hand, go a step further. They bundle the application code with a minimal pre-configured kernel, eliminating the need for a full host kernel altogether. Unikernels benefit then from a stronger protection domain (hardware), and can eliminate internal guard/permission-check functions, which increases overall application performance.

Thanks to the introduction of the [Open-Container Initiative’s image specification (OCI image spec)](https://github.com/opencontainers/image-spec) format, defining and distributing filesystems has become effortless. (In fact, the OCI image and [distribution spec](https://github.com/opencontainers/distribution-spec) have enabled a lot more than just storing and transporting applications and their filesystems!). First with DockerHub and now with open-source CNCF projects like Harbor and products like GHCR and Artifact Registry, the OCI format is has become ubiquitous in this industry for the distribution of applications.

## Unboxing Interoperability

With such standardization, it makes little sense to develop and use a new format which ultimately is aimed at serving the same purpose: the definition, storage and transportation of files representing an application. At Unikraft, we have embraced the OCI image specification format for this very purpose: to both enable distributing unikernels, as well as help define them.

To understand how we use the OCI image format, we should first consider how unikernel images can be constructed. This can be in fact done in one of two ways:

1.  By natively compiling your application against a library operating system which ultimately yields a single binary image which is capable of booting and executing on its own; or,

2.  Reading pre-compiled [ELF binary files](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) as part of a filesystem and executing these a-top of a unikernel.

Unikraft supports both approaches and more-generally uses the second as it is this which allows for interoperability with the OCI image specification. By flattening the resulting filesystem into a single tree and extracting the `ENTRYPOINT` program and executing it top of [a unikernel which supports reading and executing application binaries](https://github.com/unikraft/app-elfloader), Unikraft unikernels simply extend the specification by providing an additional file representing the unikernel.

To this end, it’s possible to build and use the resulting files from a `Dockerfile` as part of a unikernel build by supplying it as part of the `rootfs` element in a `Kraftfile` and in your repository:

```
 cmd: ["/usr/bin/my-program"] 
```

([*See more examples here*](https://github.com/kraftcloud/examples).)

## Container (Images) and Unikernels, Together

When it comes to using these two technologies, our focus at Unikraft has been to make interoperability a two-way street. To leverage existing infrastructure as much as possible in order to make unikernels approachable.

Of course, we didn’t stop at only reading images. The OCI image spec allows for the declaration of artifacts in a hierarchical fashion. The top-most element is an “index” which contains a list of “manifests”. Each manifest represents an “image” which consists of the application filesystem… and unikernel!

Thanks to the “index”, it is possible to pack applications of varying properties together. This is more commonly used to differentiate between different hardware architectures for traditional OCI container images. However, because Unikraft can target both different architectures as well as different virtual machine monitors (VMMs) and hypervisors, we enthusiastically adopted this ability in the OCI image spec and allowed for packaging unikernels into this format.

To demonstrate this, using the [open-source command-line client for Unikraft and KraftCloud, `kraft`](https://github.com/unikraft/kraftkit), you can see all the variances of a single image by platform:

```
 kraft pkg info -u unikraft.org/helloworld:latest 
```

```
 TYPE  NAME                     VERSION  FORMAT  MANIFEST  INDEX    PLAT               SIZE

app   unikraft.org/helloworld  latest   oci     6ccbe82   76f2074  linuxu/arm64       138 kB

app   unikraft.org/helloworld  latest   oci     cc9e88c   76f2074  qemu/arm64         191 kB

app   unikraft.org/helloworld  latest   oci     fc7587a   76f2074  linuxu/x86_64      93 kB

app   unikraft.org/helloworld  latest   oci     4aacacc   76f2074  xen/x86_64         143 kB

app   unikraft.org/helloworld  latest   oci     28218e9   76f2074  qemu/x86_64        225 kB

app   unikraft.org/helloworld  latest   oci     4eb44f4   76f2074  fc/x86_64          225 kB

app   unikraft.org/helloworld  latest   oci     38ac2b6   76f2074  kraftcloud/x86_64  229 kB 
```

(*These “Hello, world!” images are part of our [open-source community catalog](http://github.com/unikraft/catalog).*)

By adopting the OCI format, the technologies developed by Unikraft can be appropriately distributed and selected based on architecture, supported VMMs and hypervisors, as well as kernel-specific feature selection. This is because we also embedded KConfig options as part of the [`os.features`](https://github.com/opencontainers/image-spec/blob/main/config.md#properties) list.

The best part of this? Being able to use your existing infrastructure seamlessly.

## Containers Were Just The Beginning

At Unikraft, we’ve built KraftCloud, a new generation of cloud platform based on millisecond semantics and we make it possible to use existing container images without the overhead of actually running containers on Linux. Thanks to the performance and security of Unikraft’s technologies, the next stage of the container revolution is already here.

See how KraftCloud supports a wide range of programming languages, frameworks and applications based on `Dockerfile`s in our platform’s [application guides](https://docs.kraft.cloud/guides/) or in [the examples repository](https://github.com/kraftcloud/examples).

We’re constantly improving this and are interested to learn how you work and the problems you’re solving. Please get in touch to let us know if you don’t see your favorite app, framework, language or usecase and we’ll take a look at adding it.