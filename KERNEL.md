# Kernel

> Prerequisites:<br>
> [FS.md](FS.md)<br>
> [IX.md](IX.md)<br>

**_Disclaimer:_**<br>
*This guide is not for the faint of heart! It assumes that you have some idea of ​​what a statically linked kernel is, and how to build one for your hardware in a source-based distribution.*

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
In some distributions, you may need to run `lspci -k | grep Kernel`.

The last column is a list of modules we need. Write them down.

Next, we need to prepare a directory with the kernel sources for which we are building a config. Let's say we want to use kernel 6.15:

```shell
ix# mkdir kernel
ix# cd kernel

# get current linux kernel source
ix# wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-$(grep 6.15 $(dirname $(which ix))/pkgs/bin/kernel/6/15/ver.sh).tar.xz
ix# tar xf linux-6.15.3.tar.xz
ix# cd linux-6.15.3
```

Let's copy the old kernel configuration into our tree:

```shell
ix# cp $(dirname $(which ix))/pkgs/bin/kernel/configs/cfg_6_14 ./.config
```

Run the kernel configurator:

```shell
ix run set/menuconfig -- make HOSTCC=cc CC=cc LD=ld.ldd menuconfig
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
ix# cd linux-6.15.3
ix# ix tool reconf $(dirname $(which ix))/pkgs/bin/kernel/configs/cfg_6_14
```

Most often, to understand what needs to be included in the kernel configuration for a particular device operation, it is useful to search the web for the module name and the Gentoo/Arch link, as they have the largest knowledge base on the topic:

 * Here, for example, is a list of what needs to be done to get AMD GPU support operating - [https://wiki.gentoo.org/wiki/AMDGPU](https://wiki.gentoo.org/wiki/AMDGPU).

After configuring the kernel, copy the modified configuration to the base:

```shell
ix# cp .config $(dirname $(which ix))/pkgs/bin/kernel/configs/cfg_6_14
```

After that, you can add the kernel to the system realm in the usual way:

```shell
ix# ix mut system bin/kernel/6/15
ix# ls /bin/kernel-*
/bin/kernel-6-15
```

Remember that path, you will need it later in GRUB CLI or in grub.cfg.

Alternatively, you can use a separate realm for the bootstrap kernel:

```shell
ix# ix mut kernel bin/kernel/6/15
ix# ls /ix/realm/kernel/bin/kernel-*
/ix/realm/kernel/bin/kernel-6-15
```

Remember that path, you will need it later in GRUB CLI or in grub.cfg.

## [Legacy BIOS oddities](https://github.com/stal-ix/ix/issues/754)

In legacy BIOS, the kernel needs to have the byte sequence `aa55` at offset 510 in order for it to be bootable by GRUB. You can verify this using `hexdump -s 510 /bin/kernel-* | head -n 1`; if the output doesn't start with `00001fe aa55`, then it is not bootable and you will get `error: invalid magic number.` in GRUB. In that case, rebuild the kernel with CONFIG_EFI_STUB=n.

## Configuration options for hypervisors

### VMware
* CONFIG_FUSION=y
* CONFIG_FUSION_SPI=y
* CONFIG_HYPERVISOR_GUEST=y
* CONFIG_DRM_VMWGFX=y
* CONFIG_VMWARE_VMCI=y

<!-- Other hypervisors go here -->
