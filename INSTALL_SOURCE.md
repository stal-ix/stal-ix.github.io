# Installation from source
<sup> The stal/IX on-disk installation guide from source code </sup>

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
# gcc >= 13 can't bootstrap IX right now, so we prefer Clang where possible
test -f /usr/bin/g++ || yum install clang lld || yum install g++ || apt-get install g++
test -f /usr/bin/git || yum install git || apt-get install git
test -f /usr/bin/make || yum install make || apt-get install make
```
The exact commands for installing `parted`, `clang`/`g++`, `git`, and `make` will depend on the distribution that you use, but the above should work on Fedora, Red Hat Enterprise Linux, Debian, and their derivatives.

For general instructions on partitioning a disk, see<br>
[https://wiki.archlinux.org/title/installation_guide#Partition_the_disks](https://wiki.archlinux.org/title/Installation_guide#Partition_the_disks).<br>

Prepare EXT4 (or any other file system) on /dev/xxx using `parted` (you can also use `fdisk` or `cfdisk`), `mkfs.ext4` (or the equivalent command for your file system), and mount it:

```shell
mkdir /mnt/ix
mount /dev/xxx /mnt/ix
```

Prepare several symbolic links to form the future root filesystem:

```shell
cd /mnt/ix

ln -s ix/realm/system/bin bin
ln -s ix/realm/system/etc etc
ln -s / usr

mkdir -p home/root var sys proc dev tmp
```

Add a symbolic link to trick the **IX** package manager:

```shell
ln -s /mnt/ix/ix /ix
```

Add user "**ix**" who will own all packages on the system (note: UID 1000 is important):

```shell
useradd -ou 1000 ix
```

Prepare a managed directory owned by user **ix** in /ix, /ix/realm, etc.:

```shell
mkdir ix
chown ix ix
```

Prepare the home directory of user **ix**, owned by **ix**:

```shell
mkdir home/ix
chown ix home/ix
```

Change user to **ix** and run all commands as user **ix**:

```shell
su ix
cd /mnt/ix
```

Download the **IX** package manager, which will be used later, as the ix user before rebooting and as the root user after rebooting:

```shell
# we don't want to change our CWD
(cd home/ix; git clone https://github.com/stal-ix/ix.git)
```

Some oddities:

```shell
# like tmp dir, so realm symlink can be modified only by its creator/owner
# it is important who create/own system realm, because only they can operate it
# sudo chown {{username}} /ix/realm/system will help, iff one wants to transfer ownership 
mkdir -m 01777 ix/realm
```

And run the **IX** package manager to populate the root filesystem with bootstrap tools:

```shell
cd home/ix/ix
export IX_ROOT=/ix
export IX_EXEC_KIND=local
./ix mut system set/stalix --failsafe etc/zram/0
./ix mut root set/install
./ix mut boot set/boot/all
```

Now [prepare a bootable kernel for your hardware](KERNEL.md) and [install the GRUB bootloader](GRUB.md). Reboot into GRUB and select the menu entry corresponding to your kernel. After a successful boot, switch to tty5, the root prompt will appear.

```shell
. /etc/profile
```

We now have some useful utilities in the PATH from /ix/realm/root.

```shell
cd /home/ix/ix
# very important step, rebuild system realm
./ix mut system
```

After this, the shell will restart. In fact, after any change to the system realm, the init will restart the entire supervised process tree.

```shell
cd /home/ix/ix
./ix mut $(./ix list)
```

Rebuild the world and [add a completely new user without sudo capability](https://stal-ix.github.io/ETC#add-user).<br>

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

