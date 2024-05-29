<!--yml
category: 未分类
date: 2024-05-29 12:47:29
-->

# My Ideal Libre Computer+Phone | Matthew Peyton

> 来源：[https://mpeyton.com/posts/my_ideal_libre_computer_phone/](https://mpeyton.com/posts/my_ideal_libre_computer_phone/)

 <content>I have a very clear idea of what my “ideal” computer would be. It would incorporate all of the best aspects of the different computers (including smartphones!) I already use today, into a single machine.

## The What

First, it would have the form factor of a typical smartphone. This would allow me to carry it everywhere, and use it both one-handed and two-handed. I might even have it be a “hotdog-style foldable” to allow for comfier reading. That form factor would also allow me to stand it on a table without the use of an external kickstand.

Second, it would have powerful and open-source internals. It would have a chip equivalent in performance and efficiency to Apple’s M-series. As much RAM (> 16GB) storage (> 1TB) and battery (> 4000 mAh) that could fit in it. A USB-C port. Nothing extraneous… no gimmicks. All of it would be 100% open source, well documented, and free of black-boxes.

Third, it would run a fully free-and-open-source operating system based on GNU+Linux. The user would have full root permissions on this OS, and it would be a true Linux distro, not an Android derivative. It would use a declarative style of package management. It would be able to switch its display and window managers on-the-fly to suit whatever form factor it was being used in. Emulators and compatibility layers would allow apps and programs written for other systems to be used seamlessly.

Fourth, it would be able to “plug in” to external modules. Imagine plugging it in to a USB-C docking station at a desk. That dock is then connected to a mouse, keyboard, monitor, webcam, and external GPU. That would allow the computer to be used for coding, or graphic design. A similar dock in the form of a laptop shell could also be used. A dock near a TV could be used to turn it into a home entertainment system or video game console. VR headsets could even be connected to it. Bluetooth and WiFi would open up even more possibilities.

## The Why

This computer would solve a lot of direct problems I have with computing, as well as a lot of second-order ones.

It consolidates many devices into a single one. This removes the need to sync files and programs across devices. That, in turn, means that lots of proprietary cloud software needed for syncing can be dropped. Including phone apps that are notoriously hard to sync between devices without using a cloud intermediary. This also means that programs can be decoupled from the data they manipulate. Having the notes app just be a simple text editor on a .txt file, with no syncing required, would make taking notes a LOT simpler. (Assuming the UI/UX was adaptable to different form factors.) This benefit would extend to ALL of the other apps and programs I use. Imagine movies, books, TV shows, games, papers, documents and more. No sync necessary. Readable and writable by any program you choose.

It would also dramatically improve privacy, security and freedom. If everything on the device is FOSS, including its hardware, then everything is fully auditable. The lack of syncing also greatly increases data privacy and security. The OS and software being FOSS also gives a lot of freedom in how the device can be used. Want to run a cron job on a phone? Go ahead!

Additionally, it gives an incentive to invest more time in making the computer my own. I often don’t bother syncing all of my files, videos, games, programs, desktop customizations, tools, scripts, etc to all of my devices. Sometimes it’s not possible (e.g. You can’t run useful Python scripts on a traditional smartphone) or just a huge pain (e.g. recreating the same system color theme on a smartphone and a laptop). Using a declarative Linux OS would also greatly help with this.

Finally, its flexibility and modularity open up a lot of options. Gaming on-the-go becomes a lot better if you always have a phone-sized device with ALL your games on it. Especially if you can connect bluetooth controllers to it. Not getting enough horsepower? Just plug it in to the eGPU at your TV… and play any game you want. Or run AI models locally. Or unplug it and prop it up to watch a cooking video in the kitchen. Then later slot it into its laptop shell to get some writing done on the couch.

It might sound trivial, but I think that removing these frictions would dramatically improve the use I get out of my day-to-day computing. The fact that everything is fully open source, audited and secure would also give me immense peace of mind.

## The How

The pieces of the puzzle all already exist, but nobody has been able to put them together yet.

Apple has done a great job at unifying their various devices… but they’re all still separate hardware that you have to switch between.

[Purism](https://puri.sm/) has also done a great job at attempting to speed up the development of convergent Linux phones and other devices… but they have a long way to go. [A lot of the Linux phone operating systems and software that’s out there is promising,](https://linmob.net/) but still very early stages.

Things like folding phones are still the exclusive domain of Android OEMs for now… And [eGPUs do work on Linux](https://wiki.archlinux.org/title/External_GPU), but with a lot of caveats. USB-C docks also aren’t quite perfect for all of these use cases yet. Laptop shells exist, but only barely.

The elephant in the room is the free-and-open-source hardware. RISC-V seems to be gaining popularity… but it’s early stages and [might face some headwinds](https://cset.georgetown.edu/article/risc-v-what-it-is-and-why-it-matters/). The rest of the hardware would ALSO need to be FOSS as well. This is probably the part that’s furthest away.

## The When

I’m keeping a close eye on developments that affect the ability for this “ultimate” computer to get made. I think that while progress on the individual pieces is slow… they’re all progressing in parallel with each other. Given that, I expect it to seem hopeless for many years… until suddenly it’s not, and each individual piece reaches some critical threshold. Then it’s just a matter of integrating them together.

Remember that gaming on Linux seemed hopeless, until Proton came along. [Now 5+ years later,](https://www.gamingonlinux.com/2023/08/5-years-ago-valve-released-proton-forever-changing-linux-gaming/) gaming on Linux [surpasses gaming on Windows in many cases.](https://www.reddit.com/r/linux_gaming/comments/16vhe6a/why_does_proton_sometimes_run_games_better_than/) Incumbents always seem untouchable, and newcomers always hopeless, until suddenly it flips.

The things I’m specifically tracking are:

*   **Hardware:** [Google Pixels](https://store.google.com/us/category/phones?hl=en-US), [Pixel Fold](https://store.google.com/us/product/pixel_fold?hl=en-US&pli=1), [Purism Librem 5](https://puri.sm/products/librem-5/), [Pinephone Pro](https://pine64.org/devices/pinephone_pro/)
*   **Operating System:** [GrapheneOS](https://grapheneos.org/), [PureOS](https://www.pureos.net/), [NixOS](https://nixos.org/), [PostmarketOS](https://postmarketos.org/)
*   **Desktop Environment:** [Wayland](https://arewewaylandyet.com/), [Maui Shell](https://github.com/Nitrux/maui-shell), [GNOME](https://www.gnome.org/), [sxmo](https://sxmo.org/)
*   **Convergent Linux Software:** [Many&mldr;](https://tuxphones.com/convergent-linux-phone-apps/)
*   **Compatibility Layers:** [Wine](https://www.winehq.org/), [Proton](https://github.com/ValveSoftware/Proton), [Waydroid](https://waydro.id/), [Darling](https://www.darlinghq.org/)
*   **Docks:** [Many&mldr;](https://www.pcworld.com/article/402858/the-best-usb-c-hubs-for-your-laptop-tablet-or-2-in-1.html)
*   **eGPU:** [Many&mldr;](https://egpu.io/best-egpu-buyers-guide/)
*   **Laptop Shell:** [Nexdock](https://nexdock.com/), [other options](https://liliputing.com/5-laptop-docks-that-let-you-use-a-smartphone-like-a-notebook/)

Hopefully I can post a followup to this in a year or two to check in on how things have progressed.</content>