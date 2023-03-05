Prereq: FS.md, IX.md

stal/IX - статически слинкованный дистрибутив Linux, поэтому и 3D драйвера вкомпилированы статически.

Драйвер, с которым собирается приложение, зависит от переменной mesa_driver.

Ее можно установить на realm:

```shell
user# ix mut --mesa_driver=radeonsi
```

Тогда все приложения, собираемые в этом realm через ix mut/ix let, будут использовать выбранный драйвер.

Или для отдельного приложения:

```shell
user# ix build bin/gnome/text/editor --mesa_driver=anv
```

Как получить список доступных драйверов:

```
user# ix tool listall | grep mesa | grep drivers/
lib/mesa/drivers/anv
lib/mesa/drivers/iris
lib/mesa/drivers/llvm
lib/mesa/drivers/nouveau
lib/mesa/drivers/opengl
lib/mesa/drivers/radeonsi
lib/mesa/drivers/radv
lib/mesa/drivers/soft
lib/mesa/drivers/valve
lib/mesa/drivers/vulkan
```

Rule of thumb - если название - это имя vulkan драйвера из mesa, то, в качестве opengl драйвера будет выбран Zink - https://docs.mesa3d.org/drivers/zink.html

Поэтому, чтобы использовать zink + vulkan amd radv driver, выполните:

```shell
user# ix mut --mesa_driver=radv
```

Если у вас работает связка zink + vulkan, то она является предпочтительной, потому что компилятор шейдеров ACO существенно меньше по размеру, чем LLVM вариант.

Quirks:
* Если вы хотите использовать связку zink + vulkan, то рекомендуется добавить в ваше сессионный скрипт 
```shell
export WLR_RENDERER=vulkan # для wlroots-based композиторов
export MESA_LOADER_DRIVER_OVERRIDE=zink
```
* Intel cards operates with mesa_driver=iris, but fails with mesa_driver=anv
