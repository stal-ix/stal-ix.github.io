# Kernel

> Prerequisites:<br>
> [FS.md](FS.md)<br>
> [IX.md](IX.md)<br>

**_Disclaimer:_**<br>
*This guide is not for the faint of heart! It assumes that you have some idea of ​​what a statically linked kernel is, and how to build one for your hardware in a source-based distribution.*

*This page was last updated on January 22, 2024 (as of May 19, 2025) and may not be up to date (e.g., pkgs/bin/kernel/kernels.json was removed on [January 27, 2024](https://github.com/stal-ix/ix/commit/e167fc600108b4874887e708a09b229765445206)).*

Also you can use any suitable kernel for your hardware that:

* has MGLRU enabled
* has transparent huge pages enabled
* has cgroupsv2
* has all the usual pseudo filesystems like proc, debugfs, sysfs, devptsfs, tmpfs
* has zram
* is statically linked
* and has firmware built-in

---

This guide assumes that the IX package manager is in your PATH:

```shell
ix# export PATH=/mnt/ix/home/ix/ix:${PATH}   # assumes we are in stal/IX installer, before reboot
ix# export PATH=/home/ix/ix:${PATH}          # assumes we are in stal/IX installer, after reboot
ix# export PATH=/your/local/checkout:${PATH} # assumes local ix checkout per user
ix# ix list
```
---

The goal of this guide is to build a kernel that contains all the components needed to run it.

First, you need to know the list of modules that your hardware supports.

To do this, you can download any regular distribution with a working automatic hardware detection system.<br>
You need to do the following:

```shell
ubuntu# lspci -k
03:00.0 Class 0300: 1002:1638 amdgpu
02:00.0 Class 0108: 144d:a809 nvme
03:00.7 Class 1180: 1022:15e4 pcie_mp2_amd
03:00.5 Class 0480: 1022:15e2 snd_rn_pci_acp3x
01:00.0 Class 0280: 8086:2723 iwlwifi
03:00.3 Class 0c03: 1022:1639 xhci_hcd
03:00.1 Class 0403: 1002:1637 snd_hda_intel
00:08.1 Class 0604: 1022:1635 pcieport
00:02.4 Class 0604: 1022:1634 pcieport
03:00.6 Class 0403: 1022:15e3 snd_hda_intel
00:02.2 Class 0604: 1022:1634 pcieport
03:00.4 Class 0c03: 1022:1639 xhci_hcd
03:00.2 Class 1080: 1022:15df ccp
00:14.0 Class 0c05: 1022:790b piix4_smbus
00:08.2 Class 0604: 1022:1635 pcieport
...
```

The last column is a list of modules we need. Write them down.

Next, we need to prepare a directory with the kernel sources for which we are building a config. Let's say we want to use kernel 6.0:

```shell
ix# mkdir kernel
ix# cd kernel

# get current linux kernel source
ix# cat $(dirname $(which ix))/pkgs/bin/kernel/kernels.json | grep https
...
        "url": "https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.7.1.tar.xz",
...

ix# wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.7.1.tar.xz
ix# tar xf linux-6.7.1.tar.xz
ix# cd linux-6.7.1
```

Let's copy the old kernel configuration into our tree:

```shell
ix# cp $(dirname $(which ix))/pkgs/bin/kernel/configs/cfg_6_6_0 ./.config
```

Run the kernel configurator:

```shell
ix run set/menuconfig -- make HOSTCC=cc menuconfig
```

You need to find all the modules from the list above in the configurator (there is a search!) and add them to the configuration.

---
That being said:

 * Don't forget to add all the necessary buses for your devices (USB, I2C, PCIe, NVMe, etc.).
 * Some drivers require firmware. They'll need to be added to ix.sh for your kernel, as done here: [https://github.com/stal-ix/ix/blob/0b454258670ba4a4bc3fba6d416801d55c73467d/pkgs/bin/kernel/6/0/slot/vbox/ix.sh#L9](https://github.com/stal-ix/ix/blob/0b454258670ba4a4bc3fba6d416801d55c73467d/pkgs/bin/kernel/6/0/slot/vbox/ix.sh#L9).<br>
  *Pro tip:* Run `dmesg | grep firmware` on a running system to get information about the missing firmware!
 * Read how to build a kernel in general in a source-based distribution - [https://wiki.gentoo.org/wiki/Kernel/Configuration](https://wiki.gentoo.org/wiki/Kernel/Configuration).
 * Don't forget to add support for cgroups, user namespaces, network namespaces to your kernel!

---

Alternatively, you can combine the previous commands into one:

```shell
...
ix# cd linux-6.7.1
ix# ix tool reconf $(dirname $(which ix))/pkgs/bin/kernel/configs/cfg_6_6_0
```

Most often, to understand what needs to be included in the kernel configuration for a particular device operation, it is useful to search the web for the module name and the Gentoo/Arch link, as they have the largest knowledge base on the topic:

 * Here, for example, is a list of what needs to be done to get AMD GPU support operating - [https://wiki.gentoo.org/wiki/AMDGPU](https://wiki.gentoo.org/wiki/AMDGPU).

After configuring the kernel, copy the modified configuration to the base:

```shell
ix# cp .config $(dirname $(which ix))/pkgs/bin/kernel/configs/cfg_6_6_0
```

After that, you can add the kernel to the system realm in the usual way:

```shell
ix# ix mut system bin/kernel/6/7
ix# ls /bin/kernel-*
/bin/kernel-6-7-1...
```

Remember that path, you will need it later in GRUB CLI or in grub.cfg.

Alternatively, you can use a separate realm for the bootstrap kernel:

```shell
ix# ix mut kernel bin/kernel/6/7
ix# ls /ix/realm/kernel/bin/kernel-*
/ix/realm/kernel/bin/kernel-6-7-1-slot0
```

Remember that path, you will need it later in GRUB CLI or in grub.cfg.
