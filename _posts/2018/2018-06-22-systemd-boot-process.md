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

After the kernel is initialized, it launches systemd process. Detailed Linux boot process is described in [here](https://www.thegeekstuff.com/2011/02/linux-boot-process/). This post only handles in-systemd details.

systemd provides parallelized boot, uses sockets and d-bus activation for starting services, offers on-demand daemon launch, etc. Thus, we can easily attach our own daemons to systemd by creating service scripts in either `/lib/systemd/system` or `/etc/systemd/system` directories. For the daemons to be automatically and normally launched, we need to acknowledge the systemd launch process, which this post will investigate more.


## systemd boot process in Linux
The following chart is a structural overview of well-known systemd units and their position in the boot-up logic, according to freedesktop.
The chart comes from [here](https://www.freedesktop.org/software/systemd/man/bootup.html).

![systemd_boot](https://github.com/jonghwanchung/jonghwanchung.github.io/assets/97339878/7e348109-8513-4a45-9023-6e776de04883)



The first target of systemd to be launched is `default.target`, which is typically a symbolically linked to `graphical.target` or `multi-user.target` (depending on whether system is configured for a GUI or only a text console). If you want to add your own system service into systemd hierarchy, it would be usally be added `multi-user.target`, like for example:
(as illustrated in [https://www.freedesktop.org/software/systemd/man/systemd.target.html](https://www.freedesktop.org/software/systemd/man/systemd.target.html))

```
/etc/systemd/system/myservice.service

[Unit]
Description=My Service

[Service]
ExecStart=/usr/bin/echo 'Hello'

[Install]
WantedBy=multi-user.target
```

`timers.target`, `paths.target`, and `sockets.target` are special targets for the initialization of timers, paths, and sockets, respectively. These targets have default dependencies: target unites are automatically configured with:

- `After=sysinit.target`
- `Requires=sysinit.target`
- `Before=shutdown.target`
- `Conflicts=shutdown.target`

Those default dependencies can be ignored with the following statement in the unit: `DefaultDependencies=no`. For example, `udev.service` uses two sockets: `systemd-udevd-control.socket` and `systemd-udevd-kernel.socket`. In the chart above, `udev.service` should be initialized in the stage before `sysinit.target`. Therefore, all `udev.service`, `systemd-udevd-control.socket`, and `systemd-udevd-kernel.socket` have `DefaultDependencies=no` statement to avoid those default dependencies.

```
/lib/systemd/system/udev.service

[Unit]
Description=udev Kernel Device manager
Documentation=man:systemd-udevd.service(8) man:udev(7)
Defaultdependencies=no
Wants=systemd-udevd-control.socket systemd-udevd-kernel.socket
After=systemd-udevd-control.socket systemd-udevd-kernel.socket systemd-sysusers.service
Before=sysinit.target
...
```

```
/lib/systemd/system/systemd-udevd-control.socket

[Unit]
Description=udev Control Socket
Documentation=man:systemd-udevd.service(8) man:udev(7)
DefaultDependencies=no
Before=sockets.target
ConditionPathIsReadWrite=/sys

[Socket]
Service=systemd-udevd.service
ListenSequentialPacket=/run/udev/control    // Socket Path for FIFO UNIX domain socket
SocketMode=600
PassCredentials=yes
RemoveOnStop=yes
```

There are [more special systemd units](https://www.freedesktop.org/software/systemd/man/systemd.special.html), such like `network.target` or `network-online.target`, by default freedesktop does not show which position those targets sit in.


## What to learn next
- [systemd-analyze](https://www.freedesktop.org/software/systemd/man/systemd-analyze.html)
> - An analyze and debug system manager for boot-up performance statistics.

## References
- [bootup - System bootup process](https://www.freedesktop.org/software/systemd/man/bootup.html)
- [Systemd Boot Process a Close Look in Linux](https://linuxopsys.com/)
- [systemd.special - Special systemd units](https://www.freedesktop.org/software/systemd/man/systemd.special.html)
