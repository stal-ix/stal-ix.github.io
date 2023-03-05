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
