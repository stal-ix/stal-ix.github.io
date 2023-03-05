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

Quirks:
* Intel plugin operates with mesa_driver=iris, but fails with mesa_driver=anv
