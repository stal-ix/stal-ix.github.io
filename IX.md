# IX package manager

> Prerequisites:<br>
> [FS.md](FS.md)<br>


The **IX** package manager can be used [standalone on any supported OS](IX_standalone.md) or as the base package manager in the **stal/IX** Linux distribution.

This document describes the use of **IX** as part of **stal/IX**.

You can read the installation guide for **stal/IX** on-disk at [INSTALL.md](INSTALL.md).

## Basic concepts

A *package* is the contents of a single folder in the /ix/store directory.

For example, here are the contents of the bzip2 package:

```shell
ix# find /ix/store/0GsKotnAh74LIcvO-bin-bzip2/
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/bin
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/bin/bzip2
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/bin/bunzip2
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/bin/bzcat
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/bin/bzip2recover
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/bin/bzgrep
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/bin/bzegrep
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/bin/bzfgrep
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/bin/bzmore
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/bin/bzless
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/bin/bzdiff
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/bin/bzcmp
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/env
/ix/store/0GsKotnAh74LIcvO-bin-bzip2/touch
```

All packages form a content-addressable store, essentially similar to the same structure in NixOS and Guix.

A *realm* is also a package that contains symbolic links to other packages:

```shell
ix# find /ix/store/0Q4rkMy8J8D1WTVn-rlm-system
...
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/bin/runit
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/bin/runit-init
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/bin/runsvchdir
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/bin/utmpset
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/bin/iwctl
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/bin/iwd
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/bin/sud_client
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/bin/sud_server
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/bin/doas
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/bin/sudo
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/bin/setcwd
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/bin/mdevd
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/bin/mdevd-coldplug
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/meta.json
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/env
/ix/store/0Q4rkMy8J8D1WTVn-rlm-system/touch
```

Some realms have anchor links that mark the current (used) version of a certain realm:

```shell
ix# ls -la /ix/realm/
total 0
drwxrwxrwx .
drwxr-xr-x ..
lrwxrwxrwx boot -> /ix/store/RCa2L8DHZs71ArSI-rlm-boot
lrwxrwxrwx kernel -> /ix/store/m3K7uWjZLVDshLNq-rlm-kernel
lrwxrwxrwx pg -> /ix/store/QC6vXQZNfLfhT4t1-rlm-pg
lrwxrwxrwx system -> /ix/store/PIYCjYiLy1AIxVVl-rlm-system
```

To use the contents of a realm, simply add that realm to your PATH:

```shell
ix# export PATH="/ix/realm/boot/bin:${PATH}"
```

To make this setting happen automatically, in the first line of your session script, add:

```shell
. /etc/session
```

## Using IX

To start using **IX**, clone it from GitHub.

```shell
ix# git clone https://github.com/stal-ix/ix
ix# export PATH=${PWD}/ix:${PATH}
```

Any user with sudo configured can install packages on the system. Using a content-addressable store, different versions of packages will not overlap. Different users can use different versions of the **IX** repository. The recommended way to customize the system is to clone the repository on GitHub and make the necessary changes to your branch. Perhaps someday there will be support for overlays.

The basic command when using **IX** is `ix mut`.

Install Sway in the gui realm:

```shell
ix# ix mut gui bin/sway
```

Install Sway in the gui realm, specifying that it should use the 3D acceleration driver for AMD GPU:

```shell
ix# ix mut gui bin/sway --mesa_driver=radv
```

For a more detailed introduction to 3D acceleration in **stal/IX**, see [ACCEL.md](ACCEL.md).

Let's assume that all programs in the gui realm should use AMD GPU:

```shell
ix# ix mut gui --mesa_driver=radv
```

And remove the mesa_driver flag for software 3D:

```shell
ix# ix mut system --mesa_driver=-
```

Add a browser to the gui realm:

```shell
ix# ix mut gui bin/epiphany
```

We are tired of Sway and want to use Wayfire:

```shell
ix# ix mut gui -bin/sway bin/wayfire
```

Update all installed programs in the gui realm:

```shell
ix# ix mut gui
```

By the way, to control your named realm, you can simply exclude its name from the IX CLI:

```shell
ix# ix mut bin/telegram/desktop
ix# ix mut -bin/epiphany +bin/links
```

The command can manipulate any number of realms simultaneously. The ambiguity is eliminated by the fact that realm names cannot contain /, but package names always do:

```shell
ix# ix mut gui +bin/dosbox -bin/qemu tui +bin/links
```

Flags you specify with `--` apply to the realm if a package has not previously been specified in that realm, otherwise to the package:

```shell
ix# ix mut --mesa_driver=radv +bin/sway --mesa_driver=iris
```

With this command we specified that we need to add a flag in the user realm to use AMD GPU, but we want to use Sway with Intel GPU.

*Important!*<br>
Within a single command, all changes to a single realm occur atomically, but the anchoring of pointers to the realm itself can occur in any order.

---

*Exercise*

Explain to yourself what the following commands do:

```shell
ix# ix mut A bin/P --X=Y bin/P --X=Z -bin/P
```

```shell
ix# ix mut A -bin/P B +bin/P C +bin/P --X=Y
```

---

`ix run`

This command prepares a new realm and runs an arbitrary command in it:

```shell
ix# ix run \
    bin/qemu --for_target=aarch64-linux-user \
    bin/convert --target=linux-aarch64 \
    -- qemu-aarch64 '$(command -v convert)'
READY /ix/store/uFlUrE6DQMb3SC2l-rlm-ephemeral/touch
Version: ImageMagick 7.1.0-58 Q16-HDRI aarch64
    https://imagemagick.org
```

The example shows how to run a program built for aarch64 on x86_64 using QEMU.

`ix let`

This command does the same as `ix mut`, but does not switch the anchor link. The command is useful for checking the contents of the resulting realm before switching.

`ix build`

This is `ix let` over a temporary (ephemeral) realm. The command is useful for seeing how some set of packages (maybe just one) would look in a freshly created realm, without flags and other environments.

`ix gc`

The command finds all unused packages in /ix/store/ and moves them to the /ix/trash/ folder for asynchronous removal. A package is considered unused if there is no path to it from the anchor realms in /ix/realm/.

`ix list`

View a list of all realms or installed packages (with flags) in a specific realm.

A list of all available packages can be found at [https://github.com/stal-ix/ix/tree/main/pkgs](https://github.com/stal-ix/ix/tree/main/pkgs) or in the pkgs/ folder in your clone of the main repository.

`ix mut $(ix list)`

This command is used to update all realms.

There are a number of commands in **IX** that are implemented as separate scripts and are not part of the core. For example, because they are not well implemented as a whole or their semantics are not well developed.<br>
These commands are available through `ix tool`:

`ix tool list` - show all available commands.

Search for a wanted package by name:

```shell
ix# ix tool listall | grep ssh
lib/ssh/2
lib/ssh
bin/openssh
bin/openssh/client
bin/openssh/d
bin/openssh/d/ix
bin/tinyssh
```

Show all packages that need to be updated (uses repology.org and fuzzy search):

```shell
ix# ix tool upver
libmpc 1.3.1
courier-mta 1.2.1
sh 3.6.0
mpv-git 20221218
klipper-estimator 3.1.0
vcpkg 2022.12.14
steamos-compositor-plus 1.10.6
clipboard 0.1.3
yambar 1.9.0
libdispatch 5.7.2
```

## Advanced features

AddressSanitizer is supported for commands such as `ix build` or `ix run`.
See [ASAN.md](ASAN.md) for more details
