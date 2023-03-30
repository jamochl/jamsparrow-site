---
title: Linux Intel CPU Screen Flickering PSA
description: |
    Resolve Screen Flickering on Intel CPU systems, kernel 5.10+, and where
    to post your issues.
date: 2021-02-26
slug: linux-intel-screen-flickering-psa
categories:
    - Linux
---

![Title image tearing fix](tearing_fix.jpg)

If you have UHD 620 Intel graphics and a i5-10210U like I am, you are
likely to run into Screen flickering trouble once you upgrade to Linux
Kernel 5.10 and above. This is due to some errors in handling Intel CPU
powersaving past Linux Kernel 5.4.

If you run sudo `dmesg -l err`, you may see something like this

```
Feb 26 14:35:07 JL-Laptop kernel: i915 0000:00:02.0: [drm] *ERROR* CPU pipe A FIFO underrun
Feb 26 14:35:07 JL-Laptop kernel: i915 0000:00:02.0: [drm] *ERROR* CPU pipe B FIFO underrun
```

These cause the flickering.

The cause is described well by the user James (not myself) in the ticket of the link below.

<https://gitlab.freedesktop.org/drm/intel/-/issues/272>

> This problem seems to be related to the activity of deep package sleep
> states (on my hardware PC7 and deeper). I built and booted a 5.4 kernel
> and the problem went away, but that's only because it wasn't allowing
> package sleep states deeper than PC3.

> Check in powertop that states deeper than PC6 are actually being used
> under 5.4 and 5.8.

His solution also works to temporarily fix it as well.

> I've not found a fix, just worked around it with intel\_idle.max\_cstate=4

Essentially, the issue seems to have something to do with deeper levels of
CPU power saving being enabled by default in later kernels. The
power-saving levels are known as c-states, which you can see the
description of in this blog.

<https://www.hardwaresecrets.com/everything-you-need-to-know-about-the-cpu-c-states-power-saving-modes/>

If that blog ever goes down, just know that C0 is the CPU at full speed,
and the larger the number, ie. C4, the more power-saving is implemented
(less voltage, etc). The problem here is that not all Intel CPU's can go
past a certain C-state, if it does it will get unstable and cause the
screen flickering. Recent Linux kernel's seem to have increased the c-state
limit, which can save more power but in our case cause instability.

So, to work around this issue, you need to adjust your kernel startup
parameter to include intel\_idle.max\_cstate=4 in your GRUB or systemd-boot
config. (Change the number to whatever keeps your CPU stable, but be
mindful that lower numbers are less power efficient. Try to go as high as
you can, but 4 is a good compromise)

## For GRUB

Add `intel_idle.max_cstate=4` to the `GRUB_CMDLINE_LINUX=` variable located in
file /etc/default/grub using a text editor.

```bash
GRUB_CMDLINE_LINUX="intel_idle.max_cstate=4"
```

Save, and run

```bash
$ grub-mkconfig -o /boot/grub/grub.cfg
```

To regenerate the grub config.

## For systemd-boot

If you guys are using systemd-boot, you probably know what you are doing, if not, first find the entry you are using to boot from.

```bash
$ ls /boot/loader/entries
```

Then with a text editor (probably vim!), append

```bash
option intel_idle.max_cstate=4
```

to the end of the file.

## After

Reboot and test. to confirm that the option was loaded

```bash
$ cat /proc/cmdline
```

And you should see `intel_idle.max_cstate=4` as one of the options.

This should fix it temporarily, but it won't get fixed unless you voice out
these issues. As this is primarily a kernel issue, specifically with Intel
graphics drivers drm/i915, you should post your issues to this GitLab

<https://gitlab.freedesktop.org/drm/intel/-/wikis/How-to-file-i915-bugs>

I am expected people to see this as they migrate to Fedora 34 and Ubuntu
21.04. If you are having this problem, be sure to follow these GitLab
threads and provide your own information, so that the maintainers can
narrow down the problem.

<https://gitlab.freedesktop.org/drm/intel/-/issues/2077>

<https://gitlab.freedesktop.org/drm/intel/-/issues/272>

Keep on Linux friends!
