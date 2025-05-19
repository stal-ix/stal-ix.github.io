# Installation
<sup> The stal/IX on-disk installation guide </sup>

> Prerequisites:<br>
> [IX.md](IX.md)<br>
> [FS.md](FS.md)<br>

**_Disclaimer:_**<br>
*It is recommended that you have at least 100 GB of free storage space to bootstrap stal/IX. Expect this process to take 15 hours or more on some older and/or less powerful hardware, so be patient.*

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
```

For general instructions on partitioning a disk, see<br>
[https://wiki.archlinux.org/title/installation_guide#Partition_the_disks](https://wiki.archlinux.org/title/Installation_guide#Partition_the_disks).<br>

Prepare EXT4 on /dev/xxx using parted, mkfs.ext4, and mount it:

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

mkdir -p home/root var sys proc dev
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
./ix mut system set/stalix --failsafe --mingetty etc/zram/0
./ix mut root set/install
./ix mut boot set/boot/all
```

Now [prepare a bootable kernel for your hardware](KERNEL.md). Reboot into GRUB and run:

```shell
> linux (hdX,gptY)/boot/kernel ro root=/dev/xxx
> boot
```

where X, Y are the GRUB disk and partition numbers for /dev/xxx.
After a successful boot, switch to tty5, the root prompt will appear.

```shell
. /etc/session
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

[Bootloader](GRUB.md)<br>
[Set up Wi-Fi](WIFI.md)<br>
[Some oddities](CAVEATS.md)<br>
[System configuration](ETC.md)<br>
[User login](LOGIN.md)

<!-- {% endraw %} -->

