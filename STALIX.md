# stal/IX


This document contains a regularly updated list of differences between **stal/IX** and regular Linux.

## Minimalism

> "UNIX is simple and coherent..." - Dennis Ritchie

> "GNU's Not UNIX" -  Richard Stallman

**stal/IX** is not UNIX or Linux in the usual sense of these terms.

**stal/IX** is an attempt to rethink some fundamentals without touching the Linux API and ABI.

One of the goals of **stal/IX** is to build the system from the ground up in such a way that it is easy to understand how it works, not just easy to use.

[https://wiki.musl-libc.org/alternatives.html](https://wiki.musl-libc.org/alternatives.html)<br>
[https://github.com/illiliti/libudev-zero](https://github.com/illiliti/libudev-zero)<br>
[https://busybox.net/tinyutils.html](https://busybox.net/tinyutils.html)<br>
[https://connortumbleson.com/2022/11/28/open-source-saying-no/](https://connortumbleson.com/2022/11/28/open-source-saying-no/)

## No FHS

[https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)<br>
[FS.md](FS.md)

In general, the file system will be familiar to those who know Nix/Guix. Atomic updates, multiversioning - it's all here!

## No systemd

[https://blog.darknedgy.net/technology/2020/05/02/0/](https://blog.darknedgy.net/technology/2020/05/02/0/)<br>
[https://www.phoronix.com/news/systemd-Git-Stats-2022](https://www.phoronix.com/news/systemd-Git-Stats-2022)

Currently, **stal/IX** uses a custom init as the most lightweight solution. It formerly used runit, but it was too hard with the content-addressable store.

## Musl

[https://drewdevault.com/2020/09/25/A-story-of-two-libcs.html](https://drewdevault.com/2020/09/25/A-story-of-two-libcs.html)<br>
[https://codebrowser.dev/glibc/glibc/nptl/pthread_cancel.c.html#99](https://codebrowser.dev/glibc/glibc/nptl/pthread_cancel.c.html#99)<br>
[https://www.phoronix.com/news/Glibc-2.36-EAC-Problems](https://www.phoronix.com/news/Glibc-2.36-EAC-Problems)<br>
[https://ariadne.space/2021/12/28/glibc-is-still-not-y.html](https://ariadne.space/2021/12/28/glibc-is-still-not-y.html)

Glibc does not fully support static linking. **stal/IX** uses musl internally and allows userland software to be built with an arbitrary libc of choice.

## Non-root package management

[IX.md](IX.md)

All files on the system are owned by user IX, and all package management is performed on its behalf.

Consequently, there is not a single suid binary file in the system. Sudo is a thin layer over the local ssh daemon, to increase privileges.

## Fully supervised process tree

Every process other than init has a parent other than init. All processes that do not meet this requirement are terminated by a specially designated background process. A custom init is used to manage services, encouraging this behavior.

[https://github.com/swaywm/sway/issues/6828](https://github.com/swaywm/sway/issues/6828)<br>
[https://github.com/stal-ix/ix/blob/main/pkgs/bin/sched/stale/procs/scripts/staleprocs.sh](https://github.com/stal-ix/ix/blob/main/pkgs/bin/sched/stale/procs/scripts/staleprocs.sh)<br>
[https://unix.stackexchange.com/questions/250153/what-is-a-subreaper-process](https://unix.stackexchange.com/questions/250153/what-is-a-subreaper-process)

## Static linking

No ld.so!

[http://ewontfix.com/18/](http://ewontfix.com/18/)<br>
[https://gavinhoward.com/2021/10/static-linking-considered-harmful-considered-harmful/](https://gavinhoward.com/2021/10/static-linking-considered-harmful-considered-harmful/)<br>
[https://lore.kernel.org/lkml/CAHk-=whs8QZf3YnifdLv57+FhBi5_WeNTG1B-suOES=RcUSmQg@mail.gmail.com/](https://lore.kernel.org/lkml/CAHk-=whs8QZf3YnifdLv57+FhBi5_WeNTG1B-suOES=RcUSmQg@mail.gmail.com/)<br>
[https://eli.thegreenplace.net/2013/07/09/library-order-in-static-linking](https://eli.thegreenplace.net/2013/07/09/library-order-in-static-linking)<br>
[https://habr.com/ru/post/451208](https://habr.com/ru/post/451208)<br>
[https://lobste.rs/s/adr60v/single_binary_executable_packages](https://lobste.rs/s/adr60v/single_binary_executable_packages)<br>
[https://nullprogram.com/blog/2018/05/27/](https://nullprogram.com/blog/2018/05/27/)

## Wayland only

[http://web.archive.org/web/20231223232401/https://drewdevault.com/2021/02/02/Anti-Wayland-horseshit.html](http://web.archive.org/web/20231223232401/https://drewdevault.com/2021/02/02/Anti-Wayland-horseshit.html)

X is dying, and to keep the IX package base efficient, working on X means doing work that will one day have to be thrown away. We don't have enough resources to do that.

## Login shell

No

[https://askubuntu.com/questions/866161/setting-path-variable-in-etc-environment-vs-profile](https://askubuntu.com/questions/866161/setting-path-variable-in-etc-environment-vs-profile)

Every user session must start from the login shell, even in the ssh daemon.

[Patch for dropbear](https://github.com/stal-ix/ix/blob/main/pkgs/bin/dropbear/ix.sh#L7) to launch all processes, including non-interactive ones, with a login shell.

## Cross-compilation by default

All packages are compiled as if host platform != target platform, so we achieve that the package base is built for all platforms most of the time. We have cross-compiling CI for aarch64 and riscv!

## File associations

Existing mechanisms for associating programs with file types are complex, fragile, and difficult to integrate into IX realms.

[https://wiki.archlinux.org/title/XDG_MIME_Applications](https://wiki.archlinux.org/title/XDG_MIME_Applications)

Therefore stal/IX has its own mechanism for associating programs with file types. It is based on the [xdg-open-dispatch](https://github.com/stal-ix/ix/blob/main/pkgs/bin/xdg/open/scripts/xdg-open-dispatch) script, and changes in upstream to redirect their mechanisms to xdg-open, such as a patch for the [Epiphany web browser](https://github.com/stal-ix/ix/blob/main/pkgs/bin/epiphany/4/ix.sh#L32).

[Interaction with upstream](UPSTREAM.md)

[# TODO(pg83): cc/c++ override]: <> (This is a comment, it will not be included)
