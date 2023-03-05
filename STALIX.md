# stal/IX


This document contains a regularly replenishing list of **stal/IX** and conventional Linux differences.

## Minimalism

> "UNIX is simple and coherent..." - Dennis Ritchie

> "GNU's Not UNIX" -  Richard Stallman

**stal/IX** is not UNIX or Linux in the usual sense of these terms.

**stal/IX** - an attempt to rethink some fundamentals without touching Linux API and ABI.

One of the **stal/IX** goals - from the very beginning to build the system in such a way that it’s possible to understand how it works, and not only use it conveniently.

[https://wiki.musl-libc.org/alternatives.html](https://wiki.musl-libc.org/alternatives.html)<br>
[https://github.com/illiliti/libudev-zero](https://github.com/illiliti/libudev-zero)<br>
[https://busybox.net/tinyutils.html](https://busybox.net/tinyutils.html)<br>
[https://connortumbleson.com/2022/11/28/open-source-saying-no/](https://connortumbleson.com/2022/11/28/open-source-saying-no/)

## No FHS

[https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)<br>
[FS.md](FS.md) 

Overall, the file system will be familiar to those who know Nix/Guix. Atomic updates, multi-versioning - all here!

## No systemd

[https://blog.darknedgy.net/technology/2020/05/02/0/](https://blog.darknedgy.net/technology/2020/05/02/0/)<br>
[https://www.phoronix.com/news/systemd-Git-Stats-2022](https://www.phoronix.com/news/systemd-Git-Stats-2022)

**stal/IX** currently uses runit as the most lightweight solution, perhaps, this will change in the future.

## Musl

[https://drewdevault.com/2020/09/25/A-story-of-two-libcs.html](https://drewdevault.com/2020/09/25/A-story-of-two-libcs.html)<br>
[https://codebrowser.dev/glibc/glibc/nptl/pthread_cancel.c.html#99](https://codebrowser.dev/glibc/glibc/nptl/pthread_cancel.c.html#99)<br>
[https://www.phoronix.com/news/Glibc-2.36-EAC-Problems](https://www.phoronix.com/news/Glibc-2.36-EAC-Problems)
[https://ariadne.space/2021/12/29/glibc-is-still-not-y2038-compliant-by-default/](https://ariadne.space/2021/12/29/glibc-is-still-not-y2038-compliant-by-default/)

Glibc does not fully support static linking. **stal/IX** uses musl for internal needs, and allows to build custom soft with an arbitrary libc on a choice.

## Non-root package management

[IX.md](IX.md) 

All files on the system are IX user-owned, and all package management is done on his behalf.

Consequence - there is not a single suid binary on the system. Sudo - the thin layer over local ssh daemon, for privilege escalation.

## Fully supervised process tree

Every process different from init has a parent different from init. All processes that fail to meet this requirement are killed by a specially dedicated background process. To manage services used runit, encouraging this behavior.

[https://github.com/swaywm/sway/issues/6828](https://github.com/swaywm/sway/issues/6828)<br>
[https://github.com/stal-ix/ix/blob/main/pkgs/bin/sched/staleprocs/staleprocs.sh](https://github.com/stal-ix/ix/blob/main/pkgs/bin/sched/staleprocs/staleprocs.sh)<br>
[https://unix.stackexchange.com/questions/250153/what-is-a-subreaper-process](https://unix.stackexchange.com/questions/250153/what-is-a-subreaper-process)

## Static linking

No ld.so!

[http://ewontfix.com/18/](http://ewontfix.com/18/)
[https://gavinhoward.com/2021/10/static-linking-considered-harmful-considered-harmful/](https://gavinhoward.com/2021/10/static-linking-considered-harmful-considered-harmful/)<br>
[https://lore.kernel.org/lkml/CAHk-=whs8QZf3YnifdLv57+FhBi5_WeNTG1B-suOES=RcUSmQg@mail.gmail.com/](https://lore.kernel.org/lkml/CAHk-=whs8QZf3YnifdLv57+FhBi5_WeNTG1B-suOES=RcUSmQg@mail.gmail.com/)<br>
[https://eli.thegreenplace.net/2013/07/09/library-order-in-static-linking](https://eli.thegreenplace.net/2013/07/09/library-order-in-static-linking)<br>
[https://habr.com/ru/post/451208](https://habr.com/ru/post/451208)<br>
[https://lobste.rs/s/adr60v/single_binary_executable_packages](https://lobste.rs/s/adr60v/single_binary_executable_packages)<br>
[https://nullprogram.com/blog/2018/05/27/](https://nullprogram.com/blog/2018/05/27/)<br>

## Wayland only

[https://drewdevault.com/2021/02/02/Anti-Wayland-horseshit.html](https://drewdevault.com/2021/02/02/Anti-Wayland-horseshit.html)

X is dying, and to maintain the efficiency of the IX package base running with X means doing work that one day will have to be thrown out. We don’t have enough resources for that.

## Login shell

No<br>
[https://askubuntu.com/questions/866161/setting-path-variable-in-etc-environment-vs-profile](https://askubuntu.com/questions/866161/setting-path-variable-in-etc-environment-vs-profile)

Every user session must start from the login shell, even in ssh daemon.

[Patch for dropbear](https://github.com/stal-ix/ix/blob/main/pkgs/bin/dropbear/ix.sh#L7) to launch all processes, including non-interactive ones, with login shell.

## Cross-compile by default

All packages are compiled as if host platform != target platform, thus, we achieve that the package base is built for all platforms most of the time. We have a cross-compiling CI for aarch64 and riscv!

## File associations

Существующие механизмы привязки программ к типам файлов сложны, хрупки, и их сложно интегрировать в IX realms. https://wiki.archlinux.org/title/XDG_MIME_Applications

Поэтому в stal/IX существует свой механизм привязки программ к типам файлов. Он основан на скрипте [xdg-open-dispatch](https://github.com/stal-ix/ix/blob/main/pkgs/bin/xdg/open/scripts/xdg-open-dispatch), и на изменениях в upstream, призванных перенаправить их механизмы в xdg-open ()

## Interaction with upstream

Quite often, upstream is not interested in the ideas inherent in **stal/IX**:

* [glib developers actively hinder static linking with glib](https://bugzilla.gnome.org/show_bug.cgi?id=768215#c16);
* [VTE developers don't care about building with musl](https://gitlab.gnome.org/GNOME/vte/-/issues/72), and [don't answer questions](https://gitlab.gnome.org/GNOME/vte/-/issues/72#note_1415630);
* [musl refuses to add a preprocessor macro to determine if code is built with musl](https://wiki.musl-libc.org/faq.html);
* [sway doesn't want to patch for fully supervised process tree](https://github.com/swaywm/sway/issues/6828);
* [we can't use the execline utilities in our startup scripts because their static build is too big](https://github.com/skarnet/execline/issues/9); 
* [tty freeze after sway death](https://github.com/swaywm/sway/issues/4540), and [fix that can't enter upstream](https://github.com/stal-ix/ix/blob/main/pkgs/bin/fixtty/main.c);
* [fake dlopen, for projects that can't live without external plugins](https://github.com/pg83/dlopen); 
* [XCURSOR_SIZE support in gtk](https://github.com/stal-ix/ix/blob/main/pkgs/lib/gtk/4/stock/0.diff);
* [support for alternative database of mime types in glib](https://github.com/stal-ix/ix/blob/main/pkgs/lib/glib/ix/1.diff);
* [by default, linux kernel eat CPU cycles on breakfast](https://www.phoronix.com/news/Linux-Default-Mitigations-Off), [we disable it by default](https://github.com/stal-ix/ix/blob/main/pkgs/bin/kernel/t/2/ix.sh#L21);
* [custom gdk-pixbuf SVG loader, over lunasvg (instead of rsvg)](https://github.com/stal-ix/ix/blob/main/pkgs/lib/lunasvg/gdk/io.cpp).

Therefore, we have to maintain a set of fixes and adjustments for upstream that will never be merged into upstream.

Project goals are more important than the arrogant behavior of some maintainers!

[# TODO(pg83): cc/c++ override]: <> (This is a comment, it will not be included)
