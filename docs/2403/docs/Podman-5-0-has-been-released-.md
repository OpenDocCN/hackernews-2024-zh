<!--yml
category: 未分类
date: 2024-05-29 12:41:14
-->

# Podman 5.0 has been released!

> 来源：[https://blog.podman.io/2024/03/podman-5-0-has-been-released/](https://blog.podman.io/2024/03/podman-5-0-has-been-released/)

Podman version 5.0.0 has been released! It is our first major release in two years and includes several new features and significant changes. Podman version 5.0.0 is a very important release for Podman on Windows and Mac, featuring a complete rewrite of our code for those platforms and major improvements in hypervisor support on both platforms. Joining this are many other features, including OCI artifact support in manifests, switching to Pasta by default for rootless networking, improvements to our `containers.conf` configuration file, and many more features and fixes. Read on for details!

Podman 5’s headlining feature, and the reason we decided to create a new major release, is a complete rewrite of the `podman machine` commands. Podman machines are used to launch a Linux Virtual Machine (VM), allowing Windows and Mac systems to run Linux containers. With the rewrite, we have improved performance and stability, and significantly increased code sharing between different VM providers, making future maintenance and fixes easier. We have also added support for the Apple hypervisor on Mac, greatly improving stability, boot times, and filesharing performance on Macs. VMs managed by `podman machine` can also be easily removed with the new `podman machine reset` command. The machine rewrite will require existing users to migrate their VMs to the new backend; you can read more details on that [here](https://blog.podman.io/2024/03/migration-of-podman-4-to-podman-5-machines/).

Podman 5 also includes a number of deprecations, changes to defaults, and refinements. [Pasta](https://passt.top/passt/about/) is now the default rootless networking backend, offering greatly improved performance for rootless Podman. We’ve supported Pasta since Podman version 4.4 and now believe its performance justifies making it the default. We’ve also deprecated the BoltDB database backend and removed support for creating new Bolt databases (existing databases can still be used without issue). SQLite was made the default database for new installations in Podman version 4.9, greatly improving stability.

Podman 5 has also removed support for CNI networking on most platforms. We added Netavark, our own networking stack, in Podman 4.0, which has grown to meet or surpass CNI in all Podman use cases. We are removing CNI support due to its ongoing support burden on the team, and the plans of CNI to change their architecture in the future to focus on Kubernetes, which will prevent Podman from using it. CNI support will remain enabled in some distributions that still require it (e.g., FreeBSD and RHEL 9).

The handling of the `containers.conf` configuration file has changed, ensuring that we will never rewrite a user-modified config file – more details on that can be found [here](https://blog.podman.io/2024/03/podman-5-0-containers-conf-changes/). We’ve also made several changes to improve Docker compatibility, including small changes to the output of `podman inspect` to match that of Docker better. Finally, we have deprecated support for cgroups v1, and will remove the ability to run on systems without cgroups v2 in a future release. You can read more details about all breaking changes in [this blog](https://blog.podman.io/2024/03/podman-5-0-breaking-changes-in-detail/).

Podman version 5.0.0 also includes several other improvements. Retries for image pulls and pushes can now be controlled via the `--retry` and `--retry-delay` options. Quadlet has seen several new features, supporting template units, pods (via .pod files), and several additional keys in .container files. Finally, Podman version 5.0.0 includes dozens of bug fixes. You can find out more in the [release notes](https://github.com/containers/podman/releases/tag/v5.0.0).

.