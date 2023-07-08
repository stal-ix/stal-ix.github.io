# PKG

> This document aims to explain how to write your own build scripts but does not contain arguments for choosing this way of developing them.

ix.sh is a Jinja2 template that generates an sh script similar to PKGBUILD scripts. Therefore, it is essential to read the Jinja2 guide:<br> 
[https://jinja.palletsprojects.com/en/3.1.x/](https://jinja.palletsprojects.com/en/3.1.x/).

At the beginning of ix.sh, you need to select a template that corresponds to the package build system. You can find the list here - [https://github.com/stal-ix/ix/tree/main/pkgs/die/c](https://github.com/stal-ix/ix/tree/main/pkgs/die/c).

<!-- {% raw %} -->

```shell
{% extends '//die/c/autorehell.sh' %}
```

// at the beginning of the path indicates the path relative to the root of the packages folder. If you do not specify //, it will use the path relative to the folder with the current package.

Blocks common to all templates:

```shell
{% block fetch %}
https://xxx.yyy/zzz
sha: # specify the SHA256 checksum for the downloaded file here
{% endblock %}
```

The list of transitively inherited library targets (on the target platform) for the library is transformed into bld_libs during program build.

```shell
{% lib_deps %}
lib/c
lib/c++
{% endblock %}
```

The list of libraries used only during the build of a specific target on the target platform:

```shell
{% block bld_libs %}
lib/kernel
{% endblock %}
```

The list of libraries used to build on the host platform. Most commonly used for building code generators that need to be run at build time.

```shell
{% block host_libs %}
lib/c
{% endblock %}
```

The list of programs that can be called at build time under the host platform. Specialized build templates contain predefined lists with the necessary programs, for example, for the GNU build system, there will be make, Perl, pkg-config.

```shell
{% block bld_tool %}
bin/lz4
bld/flex
bld/bison
{% endblock %}
```

The list of programs that may be needed to run the built target:

```shell
{% block run_deps %}
bld/make
{% endblock %}
```

A block where you can set environment variables available in all build blocks:

```shell
{% block setup %}
export CFLAGS="-fcommon ${CFLAGS}"
{% endblock %}
```

A block containing instructions for patching the original source code:

```shell
{% block patch %}
patch -p0 << EOF
..some inline patch, or
{% include 'patch.diff' %}
EOF
{% endblock %}
```

The blocks of some templates are better to be supplemented rather than fully overridden:

```shell
{% block build %}
export Y=X
{{super()}}
rm some_trash_file
{% endblock %}
```

```shell
{% block install %}
{{super()}}
rm -rf ${out}/share/trash
{% endblock %}
```

You can learn about {{super()}} from the Jinja2 documentation: [https://jinja.palletsprojects.com/en/3.1.x/](https://jinja.palletsprojects.com/en/3.1.x/).

Main templates:

* autohell.sh - GNU build system without running autoreconf
* autorehell.sh - same as above, but with running autogen.sh, bootstrap.sh, or autoreconf directly

They inherit the make.sh template with all its available blocks.

Additionally, they contain a block with arguments for running configure:

```shell
{% block configure_flags %}
--with-python=python3
{% endblock %}
```

* meson.sh - Meson build system, https://mesonbuild.com/

Inherits the ninja.sh template with all its available blocks.

Additional blocks:

```shell
{% block meson_flags %}
introspection=false
{% endblock %}
```

* cmake.sh - CMAKE, https://cmake.org/

Inherit ninja.sh

Additional blocks:

```shell
{% block cmake_flags %}
SOME_COBOL_STYLE_VAR=OFF
{% endblock %}
```

* make.sh - project contains a classic Makefile

Inherits the block for building C/C++ projects. This is not entirely correct because make can describe other builds as well, but most often it is used for C/C++ projects.

Additional blocks:

```shell
{% block make_flags %}
X=Y
{% endblock %}
```

```shell
{% block make_target %}
# build targets
all
all-static
{% endblock %}
```

```shell
{% block make_install_target %}
# make install targets
install-static
install-bin-static
{% endblock %}
```

* ninja.sh - a general template for build systems that use Ninja under the hood.
<!-- {% endraw %} -->

