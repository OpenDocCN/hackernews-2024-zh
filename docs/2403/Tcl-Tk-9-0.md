<!--yml
category: 未分类
date: 2024-05-29 12:40:14
-->

# Tcl/Tk 9.0

> 来源：[https://www.tcl.tk/software/tcltk/9.0.html](https://www.tcl.tk/software/tcltk/9.0.html)

**Latest Release: Tcl/Tk 9.0b2 (May 20, 2024)**

Tcl/Tk 9.0 are now in beta testing. Those seeking its new features, or those invested in keeping their existing Tcl-related work compatible with the next releases of Tcl and Tk are invited to try and track this development work.

[Download Tcl/Tk 9.0b2 Source Releases](download.html)

### Highlights of Tcl 9.0

*   **64-bit Capacity:** Data values larger than 2Gb
*   **Unicode and Encodings:** full codepoint range, added encodings, encoding profiles to govern I/O, and more.
*   **Zip Filesystems:** mount zipfiles as filesystems
*   **Attached Archives:** enable starkit-style deployment of apps, with support data in filesystem archives attached to executable or libraries. Build tclsh and wish this way.
*   **New Notifiers:** The central event handling engine in Tcl is now constructed on top of the system calls **epoll** or **kqueue** when they are available. The **select** based implementation also remains for platforms where they are not.
*   **Many new commands and features**

### Important Incompatibilities in Tcl 9.0

*   **Namespace varname resolution:** Current namespace, not global.
*   **I/O malencoding:** now raises error by default.
*   **Tilde (~) in pathnames:** no longer interpreted as home directory.
*   **tcl_precision** no longer has effect on number formatting

### Highlights of Tk 9.0

*   **Access to OS facilities:** notifications, print, and tray systems
*   **Scalable Vector Graphics:** partial support in images, extensive use to enable scalable widget and theme appearances.
*   **Images:** full access to metadata and alpha channel.
*   **Platform Features and Conventions:** many improvements, including two-finger gesture support where available.

Download the

[Tcl/Tk 9.0b2 Release Notes](https://sourceforge.net/projects/tcl/files/Tcl/9.0b2/tcltk-release-notes-9.0b2.txt/download)

for more information.