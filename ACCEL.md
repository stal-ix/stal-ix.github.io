# 3D acceleration

> Prerequisites:<br>
> [FS.md](FS.md)<br>
> [IX.md](IX.md)

<!-- {% raw %} -->

**stal/IX** is a statically linked Linux distribution, so 3D drivers are also compiled statically.

The driver that the application is built with depends on the mesa_driver variable.

It can be installed in the realm:

```shell
user# ix mut --mesa_driver=radeonsi
```

Then all applications built in this realm using `ix mut`/`ix let` will use the selected driver.

Or for a standalone application:

```shell
user# ix build bin/gnome/text/editor --mesa_driver=anv
```

How to get a list of available drivers:

```shell
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

Rule of thumb - if the name matches the name of the Vulkan driver from Mesa, then Zink will be chosen as the OpenGL driver - [https://docs.mesa3d.org/drivers/zink.html](https://docs.mesa3d.org/drivers/zink.html).

So, to use the Zink + Vulkan AMD RADV driver, run:

```shell
user# ix mut --mesa_driver=radv
```

If zink + vulkan is a good choice for you, it is preferable because the ACO shader compiler is significantly smaller in size than the LLVM variant.

## Oddities
* If you want to use Zink + Vulkan, it is recommended to add to your session script: 
```shell
export WLR_RENDERER=vulkan # for wlroots-based composers
export MESA_LOADER_DRIVER_OVERRIDE=zink
```
* Intel cards work with mesa_driver=iris, but do not work with mesa_driver=anv.

<!-- {% endraw %} -->
