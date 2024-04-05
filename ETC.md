# etc

> Prereq:<br>
> [FS.md](FS.md)<br>
> [IX.md](IX.md)

<!-- {% raw %} -->

/etc in **stal/IX** is a symbolic link to etc/ from the system realm:

```shell
ix# ls -la /etc
lrwxrwxrwx ... /etc -> /ix/realm/system/etc
```

Files in the IX store are read only, they can't be changed. Therefore, the only way to make a setting in the installed OS **stal/IX**, is to add some package that contains the needed setting to the system realm.<br>

Most of these packages are etc/ prefixed and are located in [https://github.com/stal-ix/ix/tree/main/pkgs/etc](https://github.com/stal-ix/ix/tree/main/pkgs/etc)

*Warning:* It's important to note that, after almost any change to the system realm, runit will restart the entire process tree. Effectively, this will result in you being kicked into your login manager (emptty/mingetty/etc).

## Add user

Without sudo capability:

```shell
# cryptpw will read password from command line
root# ix mut system etc/user/0 --user={{username}} --hash=$(cryptpw)
# shell will relaunch thereafter
mkdir /home/{{username}}
chown {{username}} /home/{{username}}
```

And add sudo thereafter:

```shell
# from new user we create new RSA key pair
user$ export PATH=/ix/realm/boot/bin:$PATH # add path to bootstrapped ssh-add
user$ ssh-keygen -t rsa
...
```

```shell
# from root, cause only root can add new superusers
# assume ix in PATH, add newly crafted RSA public key
root# ix mut system etc/user/0 --pubkey="$(cat /home/{{username}}/.ssh/id_rsa.pub)"
```

## Activate zram0

```shell
ix# ix mut system etc/zram/0
```

## Remove the root console from tty5 that we added during installation

```shell
ix# ix mut system --failsafe=-
```

## Replace mingetty with emptty as login manager

*ProTip:* First try looking at [https://github.com/stal-ix/ix/blob/main/pkgs/set/system/0/unwrap/ix.sh#L18](https://github.com/stal-ix/ix/blob/main/pkgs/set/system/0/unwrap/ix.sh#L18), and come up with what the next command might look like!

*Warning:* if you don't have ~/.emptty configured, and don't have a failsafe console on tty5, then you may need a recovery.

```shell
ix# ix mut system --mingetty=- --emptty
```

## Timezone settings
The system uses UTC time by default. There is currently no global timezone setting, each user must set their own timezone in their session script:

```shell
export TZ=Europe/Moscow
```

## Setup 3D driver
[ACCEL.md](ACCEL.md)

<!-- {% endraw %} -->
