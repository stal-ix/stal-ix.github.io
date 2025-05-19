# etc

> Prerequisites:<br>
> [FS.md](FS.md)<br>
> [IX.md](IX.md)

<!-- {% raw %} -->

/etc in **stal/IX** is a symbolic link to etc/ from the system realm:

```shell
ix# ls -la /etc
lrwxrwxrwx ... /etc -> /ix/realm/system/etc
```

Files in the IX store are read-only and cannot be modified. Therefore, the only way to make a customization in an installed **stal/IX** OS is to add a package containing the desired customization to the system realm.<br>

Most of these packages have the etc/ prefix and are located at [https://github.com/stal-ix/ix/tree/main/pkgs/etc](https://github.com/stal-ix/ix/tree/main/pkgs/etc).

*Warning:* It's important to note that, after almost any change to the system realm, the init will restart the entire process tree. Effectively, this will result in you being kicked into your login manager (emptty/mingetty/etc).

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

## Remove root console from tty5 which we added during installation

```shell
ix# ix mut system --failsafe=-
```

## Replace mingetty with emptty as login manager

*Pro tip:* First try looking at [https://github.com/stal-ix/ix/blob/main/pkgs/set/stalix/unwrap/ix.sh#L32](https://github.com/stal-ix/ix/blob/main/pkgs/set/stalix/unwrap/ix.sh#L32) and imagine what the next command might look like!

*Warning:* If you don't have ~/.emptty set up and don't have a failsafe console on tty5, you may need to recover.

```shell
ix# ix mut system --mingetty=- --emptty
```

## Time zone settings
The system uses UTC time by default. There is currently no global time zone setting, each user must set their own time zone in their session script:

```shell
export TZ=Europe/Moscow
```

## Change ALSA device in sndio

```shell
ix# ix mut system --alsa_device=hw:1
```

## Setup 3D driver
[ACCEL.md](ACCEL.md)

<!-- {% endraw %} -->
