<!--yml
category: 未分类
date: 2024-05-27 15:17:04
-->

# Overview — Flent: The FLExible Network Tester

> 来源：[https://flent.org/](https://flent.org/)

# Welcome to the home of Flent

Flent is a network benchmarking tool which allows you to:

*   Easily **run network tests** composing multiple well-known benchmarking tools into aggregate, repeatable test runs.
*   **Explore** your test data through the interactive GUI and extensive plotting capabilities.
*   **Combine and aggregate** data series and produce publication quality graphs.
*   **Capture metadata** from local and remote hosts and store it along with the plot data.
*   **Collect secondary data series** such as CPU usage, WiFi, qdisc and TCP socket statistics and plot it with the main dataset.
*   Specify batch experiment runs to **completely automate your testing** regime.

Flent is written in Python and wraps well-known network benchmarking tools (such as [netperf](http://netperf.org) and [iperf](https://sourceforge.net/p/iperf2)) into aggregate, repeatable tests, such as a number of [tests for Bufferbloat](https://www.bufferbloat.net/projects/bloat/wiki/Tests_for_Bufferbloat/).

There's a [short paper (pdf)](flent-the-flexible-network-tester.pdf) and a [blog post](https://blog.tohojo.dk/2017/04/the-story-of-flent-the-flexible-network-tester.html) describing some of the design goals of Flent.

## Getting started

Install Flent as per the instructions below, then see [the Quick-start section](intro.html#quick-start) to get going.

For more information, see [the full documentation](contents.html) or [search](search.html) for specific content.

## Installing Flent

Installing Flent can be done in several ways, depending on your operating system:

*   **Debian and Ubuntu:** `apt install flent`. To install netperf, [enable the non-free repository](https://wiki.debian.org/SourcesList).
*   **Fedora:** `dnf install flent`.
*   **Gentoo:** `emerge net-analyzer/flent`.
*   **Ubuntu pre-18.04:** Add the [tohojo/flent PPA](https://launchpad.net/~tohojo/+archive/ubuntu/flent).
*   **Arch Linux:** Install Flent from [the AUR](https://aur.archlinux.org/packages/flent).
*   **FreeBSD:** `pkg install flent` to install the package or `cd /usr/ports/net/flent` to install the port.
*   **Other Linux and OSX with Macbrew:** Install from the [Python Package Index](https://pypi.python.org/pypi/flent): `pip install flent`

## Get involved

Getting involved is easy: