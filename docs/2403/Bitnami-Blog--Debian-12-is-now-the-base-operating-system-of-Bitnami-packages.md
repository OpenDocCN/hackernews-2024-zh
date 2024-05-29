<!--yml
category: 未分类
date: 2024-05-27 14:42:30
-->

# Bitnami Blog: Debian 12 is now the base operating system of Bitnami packages

> 来源：[https://blog.bitnami.com/2024/02/debian-12-is-now-base-operating-system.html](https://blog.bitnami.com/2024/02/debian-12-is-now-base-operating-system.html)

We are happy to share that we have updated the base operating system (OS) of the community edition of all Bitnami-packaged containers and Helm charts to Debian 12 (bookworm) from Debian 11 (bullseye).

This update in our containers and Helm charts helps us keep system packages more updated and reduces the number of unfixed/unpatched vulnerabilities reported by vulnerability scanners. Although we regularly update our images with the latest system packages, certain CVEs may persist until they are patched in the OS. So, changing to a newer distro will allow us to speed up the updates on our catalog. You can learn more about our CVE policy [here](https://docs.bitnami.com/kubernetes/open-cve-policy/).

Users looking to go beyond Debian and use other renowned Linux distributions as the base OS of Bitnami-packaged containers and Helm charts can leverage [Tanzu Application Catalog](https://app-catalog.vmware.com/catalog), an enterprise version of Bitnami Application Catalog. Tanzu Application Catalog offers various base images such as Debian 11 & 12, PhotonOS 4, Ubuntu 20.04 & 22.04, RedHat UBI 8 & 9, and custom-hardened golden images.

# What changes are expected?

*   **Container image tags**

*   As per the Bitnami [rolling tags policy](https://docs.bitnami.com/tutorials/understand-rolling-tags-containers/), some tags will change to reflect the new distro version i.e. *6-debian-11* will be *6-debian-12*.
*   Other rolling tags will point to the new Debian 12-based containers without an explicit mention in their name, i.e. *latest*.
*   The immutable tag will also change to show the new distro version, resetting the revision number, i.e. *6.4.1-debian-11-r4* will be *6.4.1-debian-12-r0*.
*   There would not be any deletion of any image. All the Debian 11 images will persist in the registry.

*   **bitnami/containers GitHub repository**

*   In terms of the source code, the [GitHub repository](https://github.com/bitnami/containers) will change its directory structure from *bitnami/ASSET/BRANCH/debian-11* to *bitnami/ASSET/BRANCH/debian-12*.
*   Please note both directories (debian-11 and debian-12) could coexist for some time.

*   **Helm charts**

*   Since backward compatibility won't be affected by a change in the distro version used by the containers, a major version bump of the Helm chart version is not expected.
*   A new version of every Helm chart will be released updating the bundled containers pointing to Debian 12-based images, i.e

> > repository:  bitnami/mariadb

> > - tag:  11.1.3-debian-11-r0

> > + tag:  11.1.3-debian-12-r0

> > > repository:  bitnami/os-shell
> 
> > > If you have any questions regarding this update, please feel free to get in touch with our team through
> > > 
> > > [this GitHub issue](https://github.com/bitnami/charts/issues/21362)
> > > 
> > > .