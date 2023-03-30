---
title: Arch Linux KVM/QEMU no sound fix
description: |
    If you have no sound in Guest Machines, this is why
date: 2021-02-16
slug: kvm-qemu-no-sound-fix
categories:
    - Linux
---

![Title image KVM/QEMU no sound fix](kvm-no-sound.jpg)

Just a quicky,

`spice-gtk3` has officially changed its backend from pulseaudio to
gstreamer (to account for users that don't use pulseaudio), the problem is
that they did have not bound the required gstreamer plugins as a
dependency.

Installing `gst-plugins-good` fixes the issue

```
pacman -S gst-plugins-good
```

Basically gstreamer was not able to communicate with a local sound server, `gst-plugins-good` provides the plugin that allows it to communicate with pulseaudio, hence enabling sound.

As to why it is only affecting people now I cannot say, possibly the package maintainer did not push the right build until now. If you run `spicy` in a terminal and log on to your vm, you will see in the command line a bunch of gst errors. Once you install these plugins, they go away. Don't need any fancy changes to xml or conf files!

Hope this helps!

<https://bugs.archlinux.org/task/46050>

<https://bbs.archlinux.org/viewtopic.php?id=186086>

<https://bbs.archlinux.org/viewtopic.php?id=263158>
