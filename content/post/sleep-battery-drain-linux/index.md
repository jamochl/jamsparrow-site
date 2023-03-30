---
title: Fix Linux Suspend Battery Drain
description: |
    Linux Kernel now defaults suspend to s2idle rather than deep, causing
    the sleep battery drain issue
date: 2022-01-04
slug: sleep-battery-drain-linux
categories:
    - Linux
---

After researching the following links:

<https://askubuntu.com/questions/1029474/ubuntu-18-04-dell-xps13-9370-no-longer-suspends-on-lid-close/1036122#1036122>
<https://www.kernel.org/doc/html/v4.15/admin-guide/pm/sleep-states.html>

I had discovered that the Linux Kernel post 5.4 defaults suspend to a
hardware agnostic software suspend implementation called s2idle. This is
guaranteed to work on all systems, but has the worst power efficiency. The
default for devices used to be deep.

To see if your device can support deep suspend state run this command

```
cat /sys/power/mem_sleep

# Example output
[s2idle] deep
```

If you see that the selected default is s2idle, you need to add a kernel
boot parameter to change the default back to deep.

You need to add this option

```
mem_sleep_default=deep
```

On grub, you would add this in the `/etc/default/grub` file, on
systemd-boot, you can add this in the `/boot/loader/entries/*.conf` file.
Finally on PopOS, you must use the built-in boot manager cli tool
`kernelstub`

```
sudo kernelstub -a "mem_sleep_default=deep"
```

Finally, check your `/etc/systemd/sleep.conf` file to ensure that the
`SuspendState=` has `mem` as the first option. If the commented default
indicates this, no changes need to be made.

Restart your computer, and the changes will be applied. Now you can suspend
your computer, and to check that it has successfully reached deep suspend,
run dmesg.

```
sudo dmesg | grep suspend
```

You should get the output

```
[  261.199248] PM: suspend entry (deep)
[  263.380254] PM: suspend exit
```
