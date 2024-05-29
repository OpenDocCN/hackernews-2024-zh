<!--yml
category: 未分类
date: 2024-05-27 14:49:03
-->

# How to lock away sensitive information on Linux with KDE Vaults | ZDNET

> 来源：[https://www.zdnet.com/article/how-to-lock-away-sensitive-information-on-linux-with-kde-vaults/](https://www.zdnet.com/article/how-to-lock-away-sensitive-information-on-linux-with-kde-vaults/)

DBenitostock/Getty Images

You probably have documents you want to keep away from prying eyes. These could be contracts, wills, bank account information, deeds, or just about anything that contains details about you, your family, or your life that shouldn't find their way into the wrong hands.

One of the best methods of locking those documents away is via encryption. Unfortunately, not every operating system makes it easy to manage such a task. Fortunately, if you opt to choose a Linux distribution that uses [the KDE Plasma desktop](https://www.zdnet.com/article/kde-neon-shows-that-the-plasma-6-linux-distro-is-something-truly-special/), you're in for a surprise, as encrypted folders are incredibly easy to create, thanks to KDE Vaults. 

**Also: [The first 5 Linux commands every new user should learn](https://www.zdnet.com/article/the-first-5-linux-commands-every-new-user-should-learn/)**

KDE Vaults is a built-in system that offers a user-friendly GUI to help you create encrypted vaults to lock away your more sensitive documents. With just a few clicks of the mouse, you can create an encrypted vault, ready to house all the documents (images, and other files) that shouldn't be made available to just anyone who has access to your desktop.

I'm going to walk you through the process of creating your first vault using KDE Neon, which showcases the latest desktop release, KDE Plasma 6.

## How to create your first vault

**What you'll need**: The only thing you'll need is a running instance of a Linux distribution with the KDE Plasma desktop. I would recommend using the [KDE Neon](https://www.zdnet.com/article/kde-neon-shows-that-the-plasma-6-linux-distro-is-something-truly-special/) distribution, as it includes the latest release from KDE's developers.

The fastest way to access the initial KDE Vault creation is via the notification pop-up. On your KDE Plasma panel, click the upward-pointing arrow to reveal the Status and Notifications pop-up. From there, click Vaults.

Once you've created your first Vault, you won't find the Vaults entry in the Notifications pop-up.

ZDNET/Jack Wallen

From the resulting pop-up, click Create a New Vault.

This gives you access to the Vaults creation wizard.

ZDNET/Jack Wallen

The next interactive screen requires you to name the new vault. Type a name and click Next.

You can name your vault whatever you like.

ZDNET/Jack Wallen

The next screen offers you a security notice that essentially reminds you that, while CryFS is considered safe, there has been no independent security audit done to confirm this claim. Go ahead and click Next.

You'll then be prompted to type and verify a password for the new vault. Do this and click Next.

Type and verify a password for your new vault.

ZDNET/Jack Wallen

By default, the vault will be created in the ~/.local/share/plasma-vault/NAME (where NAME is the name you've given the vault) directory, with a mount point of ~/Vaults/NAME. You can keep these defaults or change them if needed. For instance, you might want to house your vaults in a non-standard directory. Once you're okay with the location, click Next.

You can keep or change the defaults for your new vault.

ZDNET/Jack Wallen

In the next window, you can select a different cipher for the encryption, limit the vault to certain activities, and force your computer offline when the vault is open. Once you've made these configurations, click Create and your first vault will be ready.

## Using your new vault

If you click on the vaults icon in the panel (a folder with a lock), a pop-up will appear with your new vault listed. From this listing, you can either click to lock the vault (it is unlocked by default when you log in) or click the name and (from the expanded menu) open the Dolphin file manager to the new vault, forcefully lock the vault, or configure the vault.

Your new vault is listed in the Vaults pop-up menu found in the KDE Plasma panel.

ZDNET/Jack Wallen

One thing that might trip you up: Once the vault is locked, you'll still see it listed in your file manager. If you click that entry, it seems like you still have access to the folder. However, you only have access to the mount point and there will be nothing in it. The only time you can view the contents of the vault is when it is unlocked (which mounts the decrypted vault to the mount point).

**Also: [Do you need antivirus on Linux?](https://www.zdnet.com/article/do-you-need-antivirus-on-linux/)**

And that's all there is to creating and using the KDE Vaults feature. Once you have your new vault created, add all the files you need securely tucked away and then lock the vault up tight. Once locked, only those with the encryption password will be able to access the contents within.