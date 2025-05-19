# Packaging guide

> The purpose of this document is to explain how to write your own build scripts, but it does not provide any arguments for choosing this way of writing them.

ix.sh is a Jinja2 template that generates a shell script similar to PKGBUILD scripts. Therefore, it is important to read the Jinja2 manual:<br> 
[https://jinja.palletsprojects.com/en/3.1.x/](https://jinja.palletsprojects.com/en/3.1.x/).

At the beginning of ix.sh you need to select a template corresponding to the package build system. The list can be found here - [https://github.com/stal-ix/ix/tree/main/pkgs/die/c](https://github.com/stal-ix/ix/tree/main/pkgs/die/c).

<!-- {% raw %} -->

```shell
{% extends '//die/c/autorehell.sh' %}
```

// at the beginning of the path specifies the path relative to the root of the packages folder. If // is not specified, the path relative to the folder with the current package will be used.

## Template blocks

Blocks common to all templates:
```shell
{% block fetch %}
https://xxx.yyy/zzz
sha: # specify the SHA256 checksum for the downloaded file here
{% endblock %}
```

`bld_libs` is a list of libraries that are used when building a specific target on the target platform:

```shell
{% block bld_libs %}
lib/kernel
{% endblock %}
```

`lib_deps` is a list of transitively inherited libraries on the target platform that are converted to bld_libs during program build:

```shell
{% lib_deps %}
lib/c
lib/c++
{% endblock %}
```

lib/ packages should use `lib_deps`, and bin/ packages should use `bld_libs`. Sometimes, it is necessary to use `bld_libs` in lib/ packages to mitigate defects in configure scripts (which assume some existing symbols in existing libraries).

`host_libs` is a list of libraries used for building on the host platform. Most often used to build code generators that need to be run during the build:

```shell
{% block host_libs %}
lib/c
{% endblock %}
```

`bld_tool` is a list of programs that can be called during the build under the host platform. Specialized build templates contain predefined lists with the required programs. For example, for the GNU build system these would be make, Perl, pkg-config.

```shell
{% block bld_tool %}
bin/lz4
bld/flex
bld/bison
{% endblock %}
```

`run_deps` is a list of programs that may be needed to run the compiled target:

```shell
{% block run_deps %}
bld/make
{% endblock %}
```

`setup` is a block in which you can set environment variables that are available in all build blocks:

```shell
{% block setup %}
export CFLAGS="-fcommon ${CFLAGS}"
{% endblock %}
```

`patch` is a block containing instructions for patching the source code:

```shell
{% block patch %}
patch -p0 << EOF
..some inline patch, or
{% include 'patch.diff' %}
EOF
{% endblock %}
```

It is better to supplement the blocks of some templates rather than completely redefine them:

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

More information about {{super()}} can be found in the Jinja2 documentation: [https://jinja.palletsprojects.com/en/3.1.x/](https://jinja.palletsprojects.com/en/3.1.x/).

## Main templates

* autohell.sh - GNU build system without running autoreconf
* autorehell.sh - same as above, but with running autogen.sh, bootstrap.sh, or autoreconf directly

They inherit the make.sh template with all its available blocks.

In addition, they contain a block with arguments for running `configure`:

```shell
{% block configure_flags %}
--with-python=python3
{% endblock %}
```

* meson.sh - Meson build system, [https://mesonbuild.com/](https://mesonbuild.com/)

Inherits the ninja.sh template with all its available blocks.

Additional blocks:

```shell
{% block meson_flags %}
introspection=false
{% endblock %}
```

* cmake.sh - CMake, [https://cmake.org/](https://cmake.org/)

Inherits ninja.sh.

Additional blocks:

```shell
{% block cmake_flags %}
SOME_COBOL_STYLE_VAR=OFF
{% endblock %}
```

* make.sh - project contains a classic Makefile

Inherits the block for building C/C++ projects. This is not entirely correct, since make can describe other builds, but it is most often used for C/C++ projects.

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

