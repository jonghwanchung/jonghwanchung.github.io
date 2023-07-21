---
title: "systemd Boot Process"
tags:
  - research
  - linux
toc: true
toc_sticky: true
toc_label: "Table of Contents"
---

## What is _systemd_?

> "`systemd` is a suite of basic building blocks for a Linux system. It provides a system and service manager that runs as PID 1 and starts the rest of the system."
> <br>
> [https://www.freedesktop.org/wiki/Software/systemd/](https://www.freedesktop.org/wiki/Software/systemd/)


systemd is now the **init process** running as PID 1 as indicated above. `/sbin/init` was the actual init process of Linux (also known as System V init boot system). It is now replaced with `/usr/lib/systemd` in many Linux distributions.

After the kernel is initialized, it launches systemd process. Detailed Linux boot process is described below(image link). This post only handles in-systemd details.

systemd provides parallelized boot, uses sockets and d-bus activation for starting services, offers on-demand daemon launch, etc. Thus, we can easily attach our own daemons to systemd by creating service scripts in either `/lib/systemd/system` or `/etc/systemd/system` directories. For the daemons to be automatically and normally launched, we need to acknowledge the systemd launch process, which this post will investigate more.


## systemd boot process in Linux
The following chart is a structural overview of well-known systemd units and their position in the boot-up logic, according to freedesktop.
The chart comes from [here](https://www.freedesktop.org/software/systemd/man/bootup.html).

>>> Figure <<<



