---
layout: post
title: Exploiting an nVidia Optimus-Enabled Notebook with Debian Stretch
tags:
  - linux
  - programming
  - nvidia
  - notebook
---

I recently acquired a DELL Inspiron 15 7559 notebook and decided to supplement
the pre-installed Windows OS with Debian Linux (Stretch), which has been my
daily driver OS for about two years now and [will soon become][stretch] the
stable Debian version. The machine contains two GPUs&thinsp;—&thinsp;an integrated Intel 915G
GPU, and a dedicated nVidia GTX960M GPU&thinsp;—&thinsp;that are interconnected via the the
nVidia Optimus GPU switching technology.

{% include image.html url="dell-inspiron-15.png"
   description="DELL Inspiron 15 7559"
   source="http://www.pcadvisor.co.uk/review/laptops/dell-inspiron-15-7559-review-3630792/" %}

On Linux, one may either use a free [“nouveau” driver][nouveau], or the
proprietary nVidia driver. GPU computations using CUDA can only be done with
the proprietary nVidia driver and the [performance][perf] is also heavily in
nVidia's favour at the time of writing. There is, however, value to running a
completely free software stack, so the choice between the two may still pose
a dilemma to some.

Regardless of the drivers, it is necessary to deploy some system that will
switch between the two GPUs as required. On Windows, the nVidia drivers will
automatically switch between the GPUs in an attempt to minimize power
consumption when the notebook is idle. Linux, on the other hand, has a free
open-source project [Bumblebee][], which allows the user to manually run
programs using one or the other GPU. By default, the dedicated nVidia GPU is
deactivated and all processing is done by the integrated Intel GPU to save
power. When the `primusrun COMMAND` command is invoked, the dedicated GPU gets
woken by Bumblebee, and any Open GL calls made by `COMMAND` are offloaded to
the dedicated GPU. This gives the user much finer control of the hardware.

If you decide to use the nouveau driver, then all that you need to install is the
`bumblebee` package from the Debian Stretch main repository. You will also require
the nouveau driver itself (the `xserver-xorg-video-nouveau` package), but the
driver is likely to be already included in the default Debian installation. If
you decide to use the nVidia driver as I did, then you will need to install the
`bumblebee-nvidia` package from the Debian Stretch contrib repository; the
package will conveniently pull in all the other requisites including the driver
from the Debian Stretch non-free repository.

Note that the current version of `bumblebee` in the Debian Stretch repository
(3.2.1) is affected by [a bug][], which makes the notebook freeze when you run
the X server while the dedicated card is deactivated. This appears to be [a
firmware bug][] that manifests itself only when the OS has been introduced as
Windows 10 to the GPU over the ACPI protocol. To use a different OS identifier
as a workaround, insert `acpi_osi=! acpi_osi="Windows 2009"` as a kernel
parameters into your `/etc/default/grub` GRUB configuration template:

```
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="splash"
GRUB_CMDLINE_LINUX=""

# Bumblebee 3.2.1 fix (see https://github.com/Bumblebee-Project/Bumblebee/issues/764)
GRUB_CMDLINE_LINUX_DEFAULT="$GRUB_CMDLINE_LINUX_DEFAULT "'acpi_osi=! acpi_osi="Windows 2009"'
```

and generate the actual GRUB configuration file `/boot/grub/grub.cfg` by
running `sudo update-grub2`.

 [perf]: http://www.phoronix.com/scan.php?page=article&item=nvidia_2d_openclose "Linux 2D Performance: Nouveau vs. NVIDIA"
 [Bumblebee]: http://bumblebee-project.org/ "Bumblebee - NVIDIA Optimus support for Linux!"
 [a bug]: https://github.com/Bumblebee-Project/Bumblebee/issues/764 "Laptop freezes when starting X11 and discrete graphics are OFF"
 [a firmware bug]: https://bugzilla.kernel.org/show_bug.cgi?id=156341 "Nvidia fails to power on again, resulting in AML_INFINITE_LOOP/lockups (multiple laptops affected)"
 [stretch]: https://wiki.debian.org/DebianStretch "DebianStretch"
 [nouveau]: https://nouveau.freedesktop.org/wiki/ "Nouveau: Accelerated Open Source driver for nVidia cards"
