# Installation
<sup> The stal/IX on-disk installation guide </sup>

> Prereq:<br>
> [IX.md](IX.md)<br>
> [FS.md](FS.md)<br>

<!-- {% raw %} -->

Load the machine from a bootable media, such as Ubuntu/Fedora/Nix livecd, and launch the terminal:

```shell
sudo sh
```

Install the tools:

```shell
test -f /usr/bin/parted || yum install parted || apt-get install parted
test -f /usr/bin/g++ || yum install clang lld || yum install g++ || apt-get install g++ # gcc >= 13 can not boostrap IX right now, so we prefer clang where available
test -f /usr/bin/git || yum install git || apt-get install git
```

For general instructions on disk partitioning, refer to<br>
https://wiki.archlinux.org/title/installation_guide#Partition_the_disks.<br>

Prepare XFS on /dev/xxx using parted, mkfs.xfs, and mount it:

```shell
mkdir /mnt/ix
mount /dev/xxx /mnt/ix
```

Prepare some symlinks to form the future rootfs:

```shell
cd /mnt/ix

ln -s ix/realm/system/bin bin
ln -s ix/realm/system/etc etc
ln -s / usr

mkdir -p home/root var sys proc dev
```

Add a symlink to trick **IX** package manager:

```shell
ln -s /mnt/ix/ix /ix
```

Add a user "**ix**" who will own all packages in the system (note: UID 1000 is important):

```shell
useradd -u 1000 ix
```

Prepare a managed dir owned by user **ix**, in /ix, /ix/realm, etc:

```shell
mkdir ix
chown ix ix
```

Prepare the **ix** user home owned by **ix**:

```shell
mkdir home/ix
chown ix home/ix
```

Change the user to **ix** and run all commands under **ix** user:

```shell
su ix
cd /mnt/ix
```

Fetch **IX** package manager, will be used later, from ix user before reboot and by root user after reboot:

```shell
# we do not want to change our CWD
(cd home/ix; git clone https://github.com/stal-ix/ix.git)
```

Some quirks:

```shell
# like tmp dir, so realm symlink can be modified only by its creator/owner
# it is important who create/own system realm, because only they can operate it
# sudo chown {{username}} /ix/realm/system will help, iff one wants to transfer ownership 
mkdir -m 01777 ix/realm
```

And run **IX** package manager to populate the root fs with bootstrap tools:

```shell
cd home/ix/ix
export IX_ROOT=/ix
export IX_EXEC_KIND=local
./ix mut system set/stalix --failsafe --mingetty etc/zram/0
./ix mut root set/install
./ix mut boot set/boot/all
```

Now [prepare a bootable kernel for your hardware](KERNEL.md). Reboot into grub and run:

```shell
> linux (hdX,gptY)/boot/kernel ro root=/dev/xxx
> boot
```

where X, Y - GRUB disk and partition numbers for /dev/xxx. 
After a successful boot, switch into tty5, there will be a root prompt.

```shell
. /etc/session
```

Now we have some useful utilities in PATH from /ix/realm/root.

```shell
cd /home/ix/ix
# very important step, rebuild system realm
./ix mut system
```

Shell will relaunch thereafter. Actually, after any modification of the system realm, runit will reload the whole supervised process tree.

```shell
cd /home/ix/ix
./ix mut $(./ix list)
```

Rebuild the world and [add a whole new user without sudo capability](https://github.com/stal-ix/stal-ix.github.io/blob/main/ETC.md#add-user)<br>

Try logging in from tty1.

What next: 

[setup wifi](WIFI.md)<br>
[some quirks](CAVEATS.md)<br>
[system configuration](ETC.md)

<!-- {% endraw %} -->

