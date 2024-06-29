<!--yml

category: 未分类

date: 2024-05-29 12:33:29

-->

# Nova 和 staging Rust 抽象 - Danilo Krummrich

> 来源：[https://lore.kernel.org/dri-devel/Zfsj0_tb-0-tNrJy@cassiopeiae/](https://lore.kernel.org/dri-devel/Zfsj0_tb-0-tNrJy@cassiopeiae/)

```
From: Danilo Krummrich <dakr@redhat.com>
To: dri-devel@lists.freedesktop.org, nouveau@lists.freedesktop.org,
	rust-for-linux@vger.kernel.org, gregkh@linuxfoundation.org
Cc: airlied@redhat.com, lyude@redhat.com, pstanner@redhat.com,
	ajanulgu@redhat.com, mcanal@igalia.com, lina@asahilina.net,
	a.hindborg@samsung.com
Subject: Nova and staging Rust abstractions
Date: Wed, 20 Mar 2024 18:58:43 +0100	[thread overview]
Message-ID: <Zfsj0_tb-0-tNrJy@cassiopeiae> (raw)

Hi all,

In this mail I briefly want to announce the Nova project and subsequently talk about the first
efforts taken in order to upstream required Rust abstractions:

We just started to work on Nova, a Rust-based GSP-only driver for Nvidia GPUs. Nova, in the long
term, is intended to serve as the successor of Nouveau for GSP-firmware-based GPUs.

With Nova we see the chance to significantly decrease the complexity of the driver compared to
Nouveau for mainly two reasons. First, Nouveau's historic architecture, especially around nvif/nvkm,
is rather complicated and inflexible and requires major rework to solve certain problems (such as
locking hierarchy in VMM / MMU code for VM_BIND currently being solved with a workaround) and
second, with a GSP-only driver there is no need to maintain compatibility with pre-GSP code.

Besides that, we also want to take the chance to contribute to the Rust efforts in the kernel and
benefit from from more memory safety offered by the Rust programming language. Ideally, all that
leads to better maintainability and a driver that makes it easy for people to get involved into this
project.

With the choice of Rust the first problem to deal with are missing C binding abstractions for
integral kernel infrastructure (e.g. device / driver abstractions). Since this is a bit of a chicken
and egg problem - we need a user to upstream abstractions, but we also need the abstractions to
create a driver - we want to develop Nova upstream and start with just a driver stub that only makes
use of some basic Rust abstractions.

In particular, we want to start with basic device / driver, DRM and PCI abstractions and a Nova stub
driver making use of them.

Fortunately, a lot of those abstractions did already exist in various downstream trees (big thanks
to all the authors). We started picking up existing work, figured out the dependencies, fixed a few
issues and warnings and applied some structure by organizing it in the following branches.

Remotes: drm-misc [1], rust (Rust-for-Linux) [2], nova [3]

- drm-misc/topic/rust-drm:  [4] contains basic DRM abstractions, e.g. DRM device, GEM; depends on
                                 rust-device
- rust/staging/rust-device: [5] contains all common device / driver abstractions
- rust/staging/rust-pci:    [6] contains basic PCI abstractions, e.g. PCI device, but also other
                                 dependencies such as IoMem and Ressources; depends on rust-device
- rust/staging/dev:         [7] integration branch for all staging Rust abstractions
- nova/stub/nova:           [8] the nova stub driver; depends on rust-drm and rust-pci

All branches and commits are functional, but the code and commit messages are not yet in a state for
sending actual patch series. While we are working on improving those patches we would like to ask
for feedback and / or improvements already.

@Greg, can you please have a first quick look at rust-device [5]?

- Danilo

[1] https://cgit.freedesktop.org/drm/drm-misc
[2] https://github.com/Rust-for-Linux/linux
[3] https://gitlab.freedesktop.org/drm/nova
[4] https://cgit.freedesktop.org/drm/drm-misc/log/?h=topic/rust-drm
[5] https://github.com/Rust-for-Linux/linux/tree/staging/rust-device
[6] https://github.com/Rust-for-Linux/linux/tree/staging/rust-pci
[7] https://github.com/Rust-for-Linux/linux/tree/staging/dev
[8] https://gitlab.freedesktop.org/drm/nova/-/tree/stub/nova

```

* * *

```
next             reply  other threads:[~2024-03-20 17:58 UTC|newest]

Thread overview: 2+ messages / expand[flat|nested]  mbox.gz  Atom feed  top
2024-03-20 17:58 [Danilo Krummrich [this message]](#t)
2024-03-20 18:17 ` Nova and staging Rust abstractions Greg KH

```

* * *

```
Reply instructions:

You may reply publicly to this message via plain-text email
using any one of the following methods:

* Save the following mbox file, import it into your mail client,
  and reply-to-all from there: mbox

  Avoid top-posting and favor interleaved quoting:
  https://en.wikipedia.org/wiki/Posting_style#Interleaved_style

* Reply using the --to, --cc, and --in-reply-to
  switches of git-send-email(1):

  git send-email \
    --in-reply-to=Zfsj0_tb-0-tNrJy@cassiopeiae \
    --to=dakr@redhat.com \
    --cc=a.hindborg@samsung.com \
    --cc=airlied@redhat.com \
    --cc=ajanulgu@redhat.com \
    --cc=dri-devel@lists.freedesktop.org \
    --cc=gregkh@linuxfoundation.org \
    --cc=lina@asahilina.net \
    --cc=lyude@redhat.com \
    --cc=mcanal@igalia.com \
    --cc=nouveau@lists.freedesktop.org \
    --cc=pstanner@redhat.com \
    --cc=rust-for-linux@vger.kernel.org \
    /path/to/YOUR_REPLY

  https://kernel.org/pub/software/scm/git/docs/git-send-email.html

* If your mail client supports setting the In-Reply-To header
  via mailto: links, try the mailto: link

```

确保您的回复在顶部有一个**主题：**标题，并在消息正文之前有一个空行。

* * *

```
This is a public inbox, see mirroring instructions
for how to clone and mirror all data and code used for this inbox;
as well as URLs for NNTP newsgroup(s).
```
