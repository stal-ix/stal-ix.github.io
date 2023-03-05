Prereq: FS.md, IX.md

stal/IX - статически слинкованный дистрибутив Linux, поэтому и 3D драйвера вкомпилированы статически.

Драйвер, с которым собирается приложение, зависит от переменной mesa_driver.

Ее можно установить на realm:

```shell
user# ix mut --mesa_driver=radv
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

Quirks:
* Intel plugin operates with mesa_driver=iris, but fails with mesa_driver=anv
