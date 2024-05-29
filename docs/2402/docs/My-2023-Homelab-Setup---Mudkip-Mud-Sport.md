<!--yml
category: 未分类
date: 2024-05-27 14:31:10
-->

# My 2023 Homelab Setup | Mudkip Mud Sport

> 来源：[https://mudkip.me/2024/01/31/My-2023-Homelab-Setup/#Gaming-and-AI-Experiments](https://mudkip.me/2024/01/31/My-2023-Homelab-Setup/#Gaming-and-AI-Experiments)

![Homelab, Illustrated By DALL·E](img/41a77b7b45e61da53bbf7b677109c210.png)

Homelab is a place where you can store all your family’s data, self-host applications and services, locally stream media, and experiment with various technologies.

A Homelab can start with low-power devices such as a Raspberry Pi, or with hacking your router to install open-source firmware and run custom applications. It can also involve easy-to-use consumer NAS solutions like Synology. Alternatively, you might find yourself overwhelmed by used data center hardware.

Throughout 2023, I have been tinkering with my home network and computing setup, which has been both fun and rewarding.

## Hardware

Various devices operate continuously at my home. Some were shelved last year, but there are a few newcomers worth mentioning.

The router “Rotom,” which I use for my home’s internet, is an [iKoolCore R1](https://www.ikoolcore.com/products/ikoolcore-r1-pro). This compact device is powered by a Celeron N5105 processor and is equipped with 8GB of RAM, a 128GB SSD, and four 2.5GbE Ethernet ports. It runs [Proxmox VE](https://www.proxmox.com/en/proxmox-virtual-environment/overview), hosting an [OpenWRT](https://openwrt.org/) VM as my primary firewall, and a Debian VM for several networking applications.

This summer, I noticed that the SSD became read-only on several occasions due to overheating. Fortunately, the manufacturer offers an R1 Pro upgrade kit. This kit replaces the case with a heatsink that directly contacts the NVMe SSD, effectively mitigating the overheating issue. Additionally, it includes a PWM fan, which keeps this compact computer silent in my closet.

![My Network Cabinet in Augest 2023](img/deef92ffc94b82233e910658781f1aa9.png)

I have a 15U network cabinet situated in the corner of my living room. At the bottom lies a [Santak](https://www.santak.com/) TG-BOX 850 UPS, safeguarding the devices within the cabinet. Positioned in the middle is my NAS, acquired in 2020 – a [Synology DS1621+](https://www.synology.com/en-us/products/DS1621+) equipped with six 8TB HGST drives and 12GB of ECC RAM. It’s set up in a RAID-6 configuration, providing a total storage capacity of 32TB.

At the top of the cabinet is the Homelab server “Porygon,” which I assembled in 2023\. Inspired by [Tao of Mac](https://taoofmac.com/space/blog/2023/02/18/1845), it’s built around an [ASRock DeskMeet B660](https://www.asrock.com/nettop/Intel/DeskMeet%20B660%20Series/) box. The configuration includes an Intel Core i7 12700 CPU, 128GB of DDR4 RAM, two NVMe SSDs (a WD SN770 2TB and an older Samsung 970 Evo Plus 1TB), and a Geforce RTX 3060 GPU.

The DeskMeet B660 also runs Proxmox VE. The integrated Iris Xe GPU is dedicated to a [Fedora Workstation](https://fedoraproject.org/workstation/) VM for transcoding videos and offering a remote desktop. The RTX 3060 switches between a Windows VM, which I use for game streaming, and a Debian VM for doing AI experiments.

However, I’ve encountered heat issues with the “Porygon” server inside the network cabinet. The CPU, cooled by a [Noctua NH-L9i-17xx](https://noctua.at/en/nh-l9i-17xx), can quickly reach 100°C during moderately intensive tasks. The SSD, even with a heatsink, often heats up to 70°C. To address this, I implemented two solutions. Firstly, I replaced the PSU fan with a [Noctua NF-A12](https://noctua.at/en/nf-a12x25-pwm), which reduced the CPU temperature by 15°C, both at idle and under load, at the cost of [injuring both my hands](https://indieweb.social/@mudkip/111661590832909428). Secondly, I installed a dedicated cabinet fan controlled by a Xiaomi Smart Plug. When the SSD temperature hits 70°C, Home Assistant activates the fan, which turns off once the temperature drops below 65°C. Ultimately, I’m quite content with the new Homelab server. It’s usually quiet and energy-efficient, yet it can be exceptionally powerful when needed.

## Applications and Services

In my Homelab, I run a wide array of applications, catering to various needs. This includes network applications that ensure connectivity for my devices at home and allow me to access my home network remotely. I also use note-taking apps, read-later services, home automation systems, media management services, file synchronization programs, RSS readers, as well as a git server complemented by CI/CD pipelines. Moreover, I have dashboards in place to navigate and monitor all these applications efficiently. It’s fair to say that a significant portion of my digital life depends on these machines, which are entirely under my control at home.

![Homepage Dashboard and my Applications](img/94f9d90f7d9381dccc6d44ec93e1806e.png)

### On my router “Rotom”

On the compact yet capable machine, I have set up two virtual machines and one LXC container to manage various services and applications.

One VM operates as my primary router OS, running OpenWRT. I’ve configured it with passthrough for three NIC ports. The WAN port connects to the modem provided by my ISP, while the LAN ports, in conjunction with the VM net port, establish a bridged LAN network. The majority of OpenWRT’s [configuration](https://mudkip.me/2023/04/09/Project-Update-2023-Q1/#Home-Projects) revolves around DHCP settings and firewall rules.

In addition to the routing functions, “Rotom” hosts several other programs:

*   **[Jellyfin](https://github.com/jellyfin/jellyfin)**: This serves as my secondary media center. I’m contemplating a switch from Plex to Jellyfin since it is fully open-source and doesn’t depend on a remote server.
*   **[frpc](https://github.com/fatedier/frp)**: This is for exposing certain services, like an SSH service that enables me to connect back to my home network. It’s useful because I have a CG-NAT IPv4 network. If I encounter a lack of IPv6 support in an external Wi-Fi network, I rely on frp to access my home network.
*   **[cloudflared](https://github.com/cloudflare/cloudflared)**: This acts as an alternative method for exposing services via Cloudflare Tunnels.
*   **[cloudflare-ddns](https://github.com/timothymiller/cloudflare-ddns)**: A dynamic DNS service that maintains an AAAA record, facilitating direct IPv6 connections back to my home.
*   **[traefik](https://github.com/traefik/traefik)**: This reverse proxy manages services not running on “Porygon.” I’ll detail the services running on the Homelab PC separately.

Jellyfin operates within an LXC container, with the Intel iGPU passed through for video transcoding. The remaining services are housed in Docker containers running on a Debian VM. The Debian VM also functions as a *side router* for my Homelab PC and devices such as the Apple TV and Sonos One.

### On my Synology NAS “Uxie”

As my storage center, I need my NAS to be as robust as possible, so I haven’t installed too many applications on it.

In addition to standard NAS services like SMB, NFS, and File Station, the applications I use include:

*   **[Synology Photos](https://www.synology.com/en-global/dsm/feature/photos)**: Backs up the photos of my family. I’m considering switching to an open-source solution like Immich, but the last time I tried, Immich didn’t integrate well with the iCloud Photo Library.
*   **[qBittorrent](https://github.com/qbittorrent/qBittorrent)** and **[VueTorrent](https://github.com/VueTorrent/VueTorrent)**: Mostly used to download *Linux ISOs*.
*   **[aria2](https://github.com/aria2/aria2)**: Another tool for downloading *Linux ISOs*.
*   **[Forgejo](https://codeberg.org/forgejo/forgejo)**: A self-hosted Git server; it’s lightweight and a community fork of Gitea.
*   **[Sonatype Nexus Repository](https://www.sonatype.com/products/sonatype-nexus-oss-download)**: My private package manager for Docker and npm, and also a cache proxy for [docker.io](http://docker.io) and [ghcr.io](http://ghcr.io) to ensure all my applications can start without an internet connection.
*   **[MinIO](https://github.com/minio/minio)**: An S3-like object storage service for my other applications.
*   **[Navidrome](https://github.com/navidrome/navidrome)**: A lightweight music manager compatible with the Subsonic API.
*   **[Sonarr](https://github.com/Sonarr/Sonarr)**: An automatic media manager for anime and TV shows.
*   **[Radarr](https://github.com/Radarr/Radarr)**: An automatic media manager for movies.
*   **[Bazarr](https://github.com/morpheus65535/bazarr)**: A subtitle downloader for content in Sonarr and Radarr.

Apart from first-party Synology apps, all third-party programs are running in Docker containers.

### A Home Kubernetes Cluster in “Porygon”

I’ve migrated the [k3s](https://github.com/k3s-io/k3s) control plane and worker VMs from Frost Canyon NUC “Comet” to my new Homelab PC. Most of my applications are running in this k3s cluster because it’s easy to maintain, upgrade, and recover. Building this has also been a great way to learn.

Here’s a list of applications running in this cluster:

*   **[Memos](https://github.com/usememos/memos)**: My primary note-taking app for capturing my thoughts and ideas. I also developed the iOS and Android client titled “[Moe Memos](https://memos.moe/).”
*   **[Miniflux](https://github.com/miniflux/v2)**: My RSS reader. It supports the Fever and Google Reader API and can be connected to various RSS clients.
*   **[Vaultwarden](https://github.com/dani-garcia/vaultwarden)**: An unofficial server written in Rust for the open-source password manager Bitwarden. It’s more lightweight than the official server.
*   **[Wallabag](https://github.com/wallabag/wallabag)**: My link manager and “read later” app.
*   **[MicroBin](https://github.com/szabodanika/microbin)**: A text and file-sharing application for when I’m transferring things without Universal Clipboard.
*   **[NocoDB](https://github.com/nocodb/nocodb)**: An AirTable-like smart spreadsheet. I use it for personal project management and to track games and movies.
*   **[Komga](https://github.com/gotson/komga)**: My manga library. It supports OPDS clients like Panels and Mihon.
*   **[Paperless-ngx](https://github.com/paperless-ngx/paperless-ngx)**: A document management system for organizing scanned documents.
*   **[Stirling PDF](https://github.com/Stirling-Tools/Stirling-PDF)**: An all-in-one PDF toolkit.
*   **[Audiobookshelf](https://github.com/advplyr/audiobookshelf)**: I use it as an automatic podcast downloader.
*   **[Photoview](https://github.com/photoview/photoview)**: I use it to manage collected pictures *not* in my photo library, such as CD and Blu-ray booklets.
*   **[Prowlarr](https://github.com/Prowlarr/Prowlarr)** backed by **[FlareSolverr](https://github.com/FlareSolverr/FlareSolverr)**: An indexer manager for Sonarr/Radarr.
*   **[YoutubeDL-Material](https://github.com/Tzahi12345/YoutubeDL-Material)**: Downloads videos from YouTube via [yt-dlp](https://github.com/yt-dlp/yt-dlp).
*   **[draw.io](https://github.com/jgraph/drawio)**: The most powerful diagramming tool.
*   **[Kimai](https://github.com/kimai/kimai)**: A self-hosted time tracker.
*   **[Firefly III](https://github.com/firefly-iii/firefly-iii)**: A self-hosted finance manager. I haven’t used it much yet.
*   **[Node-RED](https://github.com/node-red/node-red)** and **[n8n](https://github.com/n8n-io/n8n)**: Powerful automation tools, but I haven’t used them much yet.
*   **[Woodpecker CI](https://github.com/woodpecker-ci/woodpecker)**: A Kubernetes native continuous integration solution.

The manifest of my Kubernetes cluster is managed in a Git repository and is automatically deployed via a GitOps tool named [Flux CD](https://github.com/fluxcd/flux2). When I push changes to the repository, such as adding a new application or upgrading Docker images, the deployment occurs within a few minutes.

Recently, I discovered a powerful tool named [Renovate Bot](https://github.com/renovatebot/renovate). It scans my Kubernetes manifest repository every few hours to identify which Helm releases or container image tags require upgrades and then creates pull requests in Forgejo, similar to how Dependabot operates on GitHub.

In addition to the applications I’m running, my setup includes the [Ingress NGINX Controller](https://github.com/kubernetes/ingress-nginx), [MetalLB](https://github.com/metallb/metallb), and [cert-manager](https://github.com/cert-manager/cert-manager) to facilitate service access and manage certificates. The data for these applications is stored on [Longhorn](https://github.com/longhorn/longhorn) volumes, which are replicated across all three worker VMs and two SSDs. I’ve also integrated [Kube Prometheus Stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) and [Grafana Loki](https://github.com/grafana/loki) to monitor the cluster and review pod logs.

Another tool I use in parallel with `kubectl` for managing resources in the cluster is [Portainer](https://github.com/portainer/portainer). I’ve also configured Portainer environments to manage Docker Compose stacks on my NAS, router, and several cloud servers. For convenient navigation through all the apps in my Homelab, I utilize a highly customizable dashboard named [Homepage](https://github.com/gethomepage/homepage). It seamlessly integrates with Kubernetes and is capable of discovering services based on Ingress annotations.

All k3s nodes operate under Ubuntu 22.04, and I’ve configured the [system-upgrade-controller](https://github.com/rancher/system-upgrade-controller) to automatically update k3s and apt.

### Remote Development and Desktop

Like my Kubernetes cluster, The Fedora Workstation VM “comet-core” is also migrated from the Frost Canyon NUC. The Intel Iris Xe iGPU is passed through it. I use it mostly as a remote development server, and when I need GUI applications, I can access the [GNOME desktop](https://www.gnome.org/) with RDP via [xrdp](https://github.com/neutrinolabs/xrdp) and [xorgxrdp](https://github.com/neutrinolabs/xorgxrdp).

Notable applications running in this VM contains:

*   **[Plex](https://www.plex.tv/)**: My primary media library. I have lifetime Plex Pass and I still prefer Plex for a variety of unique features. But I’m kinda worried about Plex’s [future](https://techcrunch.com/2023/01/05/streamer-plex-finally-ready-to-launch-a-tv-and-movie-rentals-marketplace/) and its [troublesome handling of privacy](https://www.techhive.com/article/2157803/plex-discover-together-privacy-concerns.html). Maybe I’ll switch to Jellyfin some day.
*   **[code-server](https://github.com/coder/code-server)**: A web development environment. However, I often develop on this VM via the [Remote Development](https://code.visualstudio.com/docs/remote/remote-overview) feature in Visual Studio Code.
*   **[calibre](https://calibre-ebook.com)** and **[calibre-web](https://github.com/janeczku/calibre-web)**: My book library. I also use calibre to DeDRM and convert digital books I bought.
*   **[Handbrake](https://handbrake.fr/)**: I use it to transcode videos from my Blu-ray collection to H.265 with Intel Quick Sync.

### Gaming and AI Experiments

Having a dedicated GPU has significantly justified upgrading my Homelab PC. I’ve set up a Windows VM “Porygon-Z” and a Debian VM “Porygon2,” both configured to utilize the RTX 3060 GPU. However, only one VM can be powered on at a time since GPU passthrough is restricted to a single VM.

In the Debian VM, I’ve been experimenting with a few smaller LLM apps such as [Ollama](https://ollama.ai/) and [ChatGLM-6B](https://github.com/THUDM/ChatGLM-6B/blob/main/README_en.md), as well as creating art with the [Stable Diffusion web UI](https://github.com/AUTOMATIC1111/stable-diffusion-webui). I’ve also explored vector databases such as [Chroma](https://github.com/chroma-core/chroma).

On the Windows VM, I’ve installed [Sunshine](https://github.com/LizardByte/Sunshine), primarily to stream games from PC Game Pass, and titles that are either unsupported or don’t perform well on the Steam Deck, to my handheld console with [Moonlight](https://moonlight-stream.org/). The ability to play games at 1080p 60fps with the highest settings on a handheld device was a dream I never thought would be possible until now.

### Other VMs and LXC Containers

Last but not least, I’d like to highlight more applications.

[Nextcloud](https://nextcloud.com/) serves as my file synchronization server. It used to perform poorly, particularly when syncing a large number of small files. However, with recent upgrades to both Nextcloud versions and my hardware, its performance has significantly improved, offering super-fast file transfers. I’ve set up Nextcloud with [Nextcloud AIO](https://github.com/nextcloud/all-in-one) in an LXC container on “Porygon.” Beyond file syncing, Nextcloud also manages my calendar and tasks through CalDAV and syncs browser bookmarks using [Floccus](https://floccus.org/).

[Home Assistant OS](https://www.home-assistant.io/) orchestrates all my IoT devices and bridges them to HomeKit. For devices that don’t natively support HomeKit, I can access them through [hass-xiaomi-miot](https://github.com/al-one/hass-xiaomi-miot). Home Assistant also automates various tasks, such as controlling the network cabinet fan I mentioned earlier and activating the Aqara M1S night light when someone enters the living room at night.

## Backups

I loosely adhere to the 3-2-1 backup strategy for safeguarding my data. My NAS serves as the primary backup target for various devices, including my family’s iMac via Time Machine and two Proxmox VE devices using a [Proxmox Backup Server](https://www.proxmox.com/en/proxmox-backup-server/overview) [VM](https://www.synology.com/en-global/dsm/feature/virtual_machine_manager). Additionally, my older Synology DS916+ NAS, which has been operational for 8 years, secures the data from my main NAS through [rsync](https://kb.synology.com/en-us/DSM/help/DSM/AdminCenter/application_backupserv_sharedfoldersync?version=7) and [Hyper Backup](https://www.synology.com/en-global/dsm/feature/hyper_backup).

For offsite backup, I utilize [Backblaze B2](https://www.backblaze.com/cloud-storage) and OneDrive. Backblaze B2 houses backups from Hyper Backup, [restic](https://github.com/restic/restic), Longhorn, and [Arq](https://www.arqbackup.com/), while selected folders from my NAS are one-way synced to OneDrive using [Cloud Sync](https://www.synology.com/en-global/dsm/feature/cloud_sync). I’ve opted not to have a remote backup for my media library due to the storage costs, considering that most of the content can be reacquired by re-ripping physical media or re-downloading.

## Homelab in 2024

That’s all for my 2023 Homelab setup. I’m very satisfied with my current setup, and there are a few areas I’d like to explore.

Currently, I only have Gigabit Ethernet, even though I have a few 2.5GbE devices. Perhaps I should add a few 2.5GbE switches to access my data faster.

I’d like to make the most of my devices. I’m considering reconfiguring my Frost Canyon NUC as a backup device in case one or two pieces of hardware in my Homelab fail. It happened before with the Noctua CPU fan, and I lost access to “Porygon” for a few days. I also have a spare [Raspberry Pi 4](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/) and a [Zero 2 W](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/), but I haven’t figured out how to utilize them effectively.

I’m also interested in open-source NAS solutions such as [mergerfs](https://github.com/trapexit/mergerfs), [SnapRAID](https://github.com/amadvance/snapraid), and [openmediavault](https://github.com/openmediavault/openmediavault), but I’m not yet motivated to build one. I’m looking into VLAN setup as well, although I don’t think I really need it yet.

Overall, I have enjoyed my Homelab journey and am looking forward to more building and learning. I hope you enjoyed reading this post, and if you have any suggestions or ideas, please reach out to me on [Mastodon](https://indieweb.social/@mudkip).