# stal/IX Filesystem


> This document describes the structure of the **stal/IX** filesystem.

<!-- {% raw %} -->

/ix/store/ contains a set of folders, each package corresponding to one folder.<br>
The folders form a content-addressable store, i.e. all paths uniquely identify a unique package.

```shell
pg-> ls -la /ix/store/ | head -n 10
total 340
drwxr-xr-x 5664 ix       1000        393216 Dec 11 16:46 .
drwxr-xr-x    6 ix       root            58 Nov  4 23:02 ..
drwxr-xr-x    4 ix       1000            56 Dec  7 01:17 00BoJm6qe56myzQk-lib-jxl
drwxr-xr-x    4 ix       1000            56 Dec  5 19:48 00OKaSNZ7iL7AJWm-lib-ssh-2
drwxr-xr-x    4 ix       1000            56 Dec  7 02:37 01JxssFZ9igzTy5i-lib-web-kit-gtk-orig
drwxr-xr-x    4 ix       1000            54 Dec  7 02:02 01S601SnGrScamy9-bin-xchm-unwrap
drwxr-xr-x    2 ix       1000            50 Dec  1 14:06 02P6wVvlRjp7YoRD-url-libxslt-v1-1-37-tar-bz2
drwxr-xr-x    3 ix       1000            41 Dec 11 04:10 02TsF9yb8tvNU18u-bin-p7zip-a
drwxr-xr-x    2 ix       1000            44 Dec  1 14:06 03xrH1zQOmgF5IBK-lnk
```

The /ix/realm/ folder contains pointers to folders from /ix/store/:

```shell
pg-> ls -la /ix/realm/
total 0
drwxrwxrwx    2 ix       1000            56 Dec 11 16:46 .
drwxr-xr-x    6 ix       root            58 Nov  4 23:02 ..
lrwxrwxrwx    1 pg       10000           35 Dec 11 06:08 boot -> /ix/store/8V4bablsXQcSK6ZY-rlm-boot
lrwxrwxrwx    1 pg       10000           37 Dec 11 06:08 kernel -> /ix/store/GAw71z9yAtwOStPd-rlm-kernel
lrwxrwxrwx    1 pg       10000           33 Dec 11 16:46 pg -> /ix/store/w5qTNK0MpREVL4Cy-rlm-pg
lrwxrwxrwx    1 pg       10000           37 Dec 11 06:08 system -> /ix/store/oQfJCY3xa3jlPkNf-rlm-system
```

In fact, these are the "roots" that the **IX** package manager can use to figure out what is actively used in /ix/store/ and what can be safely removed using the `ix gc` command.

Some realms are predefined:

/ix/realm/system - the "system" realm, contains the code necessary to boot the OS.

The root folders /etc, /bin look at the system realm:

```shell
pg-> ls -la /bin /etc
lrwxrwxrwx    1 root     root            20 May 22  2022 /bin -> /ix/realm/system/bin
lrwxrwxrwx    1 root     root            20 May 22  2022 /etc -> /ix/realm/system/etc
```

By the way, this is why there is no point in editing files in /etc, they will be updated with any update of /ix/realm/system.

For each user on the system named USER, there is a realm /ix/realm/USER that belongs to that user:

* It is the default when using `ix mut`: `ix mut bin/nano` will install nano in your personal realm.
* It comes first in PATH:

```shell
pg-> echo ${PATH}
/ix/realm/pg/bin:/bin
```

For this to work, your session manager must do `. /etc/session`.

<!-- {% endraw %} -->
