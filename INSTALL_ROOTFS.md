# Installation from rootfs
<sup> The stal/IX on-disk installation guide from a rootfs tarball </sup>

> Prerequisites:<br>
> [IX.md](IX.md)<br>
> [FS.md](FS.md)<br>

<!-- {% raw %} -->

Boot the machine from a bootable media, such as an Ubuntu/Fedora/NixOS live CD, and launch a terminal:

```shell
sudo sh
```

Install the tools:

```shell
test -f /usr/bin/parted || yum install parted || apt-get install parted
test -f /usr/bin/wget || yum install wget2 || apt-get install wget
test -f /usr/bin/tar || yum install tar || apt-get install tar
test -f /usr/bin/xz || yum install xz || apt-get install xz-utils
```
The exact commands for installing `parted`, `wget`, `tar`, and `xz` will depend on the distribution that you use, but the above should work on Fedora, Red Hat Enterprise Linux, Debian, and their derivatives.

For general instructions on partitioning a disk, see<br>
[https://wiki.archlinux.org/title/installation_guide#Partition_the_disks](https://wiki.archlinux.org/title/Installation_guide#Partition_the_disks).<br>

Prepare EXT4 (or any other file system) on /dev/xxx using `parted` (you can also use `fdisk` or `cfdisk`), `mkfs.ext4` (or the equivalent command for your file system), and mount it:

```shell
mkdir /mnt/ix
mount /dev/xxx /mnt/ix
```

Download the latest stal/IX rootfs tarball from [here](https://github.com/stal-ix/stalix/releases/latest) (e.g. https://github.com/stal-ix/stalix/releases/download/20250627/stalix-x86_64-20250627.tar.xz) and extract it:

```shell
cd /mnt/ix

wget https://github.com/stal-ix/stalix/releases/download/20250627/stalix-x86_64-20250627.tar.xz
tar -xpJf stalix-*-*.tar.xz
```

And `pivot_root` inside of the root filesystem (`chroot` doesn't work with `unshare` used by **IX**):

```shell
mkdir old-root
pivot_root . old-root
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount --rbind old-root/dev /dev
mount -t tmpfs -o mode=1777 tmpfs /dev/shm
cp old-root/etc/resolv.conf /var/run/resolvconf/
. /etc/profile
```

Now [prepare a bootable kernel for your hardware](KERNEL.md) and [install the GRUB bootloader](GRUB.md). Reboot into GRUB and select the menu entry corresponding to your kernel. After a successful boot, switch to tty5, the root prompt will appear.

```shell
. /etc/profile
```

[Add a completely new user without sudo capability](https://stal-ix.github.io/ETC#add-user).<br>

Try logging in from tty1.

What's next:

Add uniqueness to the system, without it some packages refuse to install:

```shell
./ix mut system --seed="$(cat /dev/random | head -c 1000 | base64)"
```

[Set up Wi-Fi](WIFI.md)<br>
[Some oddities](CAVEATS.md)<br>
[System configuration](ETC.md)<br>
[User login](LOGIN.md)

<!-- {% endraw %} -->

