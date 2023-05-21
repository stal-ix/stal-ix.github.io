<!-- {% raw %} -->

ix.sh представляют из себя jinja2 шаблон, из которого генерируется sh script, в чем-то похожий на скрипты PKGBUILD. Поэтому обязательно прочтите гайд по jinja2! https://jinja.palletsprojects.com/en/3.1.x/

Данный документ призван рассказать, как писать свои сборочные скрипты, но не содержит аргументации за выбор такого способа их разработки.

В самом начале ix.sh нужно выбрать шаблон, соответствующий системе сборки пакета. Их список можно подсмотреть тут - https://github.com/stal-ix/ix/tree/main/pkgs/die/c

```shell
{% extends '//die/c/autorehell.sh' %}
```

Блоки, общие для всех шаблонов:

```shell
{% block fetch %}
https://xxx.yyy/zzz
sha: # тут идет указание sha256 checksum для скачанного файла
{% endblock %}
```

Список транзитивно наследуемых библиотечных целей(под target platform) для библиотеки, превращается в bld_libs при сборке программы.
```
{% lib_deps %}
lib/c
lib/c++
{% endblock %}
```

Список библиотек, используемых только при сборке какой-то цели, под target platform:
```shell
{% block bld_libs %}
lib/kernel
{% endblock %}
```

Список библиотек, используемых при сборке, под host platform. Чаще всего используется для сборки кодогенераторов, которые надо запустить во время сборки.
```shell
{% block host_libs %}
lib/c
{% endblock %}
```

Список программ, которые могут вызваны во время сборки, под host platform. Специализированные сборочные шаблоны содержат заранее определенные списки с нужными программами, например, для GNU build system там будет make, perl, pkg-config
```shell
{% block bld_tool %}
bin/lz4
bld/flex
bld/bison
{% endblock %}
```

Список программ, которые могут понадобиться для запуска построенной цели:
```shell
{% block run_deps %}
bld/make
{% endblock %}
```

Блок, где можно выставить переменные среды, доступные во всех сборочных блоках:
```shell
{% block setup %}
export CFLAGS="-fcommon ${CFLAGS}"
{% endblock %}
```

Блок, содержащий инструкции для патчинга оригинальных исходников:
```shell
{% block patch %}
patch -p0 << EOF
..some inline patch, or
{% include 'patch.diff' %}
EOF
{% endblock %}
```

Блоки некоторых шаблонов имеет смысл не переопределять полностью, а дополнять:

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

А что такое {{super()}}, вы можете узнать из документации на jinja2! https://jinja.palletsprojects.com/en/3.1.x/

Основные шаблоны:

* autohell.sh - GNU build system, без запуска autoreconf
* autorehell.sh - то же самое, но с запуском autogen.sh, bootstrap.sh, или autoreconf напрямую

Наследуют шаблон make.sh, со всеми доступными из него блоками.

Дополнительно содержат блок с аргументами для запуска configure:
```shell
{% block configure_flags %}
--with-python=python3
{% endblock %}
```

* meson.sh - meson build system, https://mesonbuild.com/

Наследуют шаблон ninja.sh, со всеми доступными из него блоками.

Дополнительные блоки:

```shell
{% block meson_flags %}
introspection=false
{% endblock %}
```

* cmake.sh - CMAKE, https://cmake.org/

Наследуют ninja.sh

Дополнительные блоки:

```shell
{% block cmake_flags %}
SOME_COBOL_STYLE_VAR=OFF
{% endblock %}
```

* make.sh - проект содержит классический Makefile

Наследует блок для сборки C/C++ проектов. Это не очень правильно, потому что на make можно описать и другую сборку, но, чаще всего, это именно C/C++ проект.

Дополнительные блоки:

```shell
{% block make_flags %}
X=Y
{% endblock %}
```

```shell
{% block make_target %}
# цели для сборки
all
all-static
{% endblock %}
```

```shell
{% block make_install_target %}
# цели для make install
install-static
install-bin-static
{% endblock %}
```

* ninja.sh - общий шаблон для систем сборки, использующих Ninja под капотом.
<!-- {% endraw %} -->
