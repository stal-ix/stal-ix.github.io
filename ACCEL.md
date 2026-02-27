# 3D acceleration

> Prerequisites:<br>
> [FS.md](FS.md)<br>
> [IX.md](IX.md)

<!-- {% raw %} -->

**stal/IX** is a statically linked Linux distribution, so 3D drivers are also compiled statically.

The driver is selected by specifying --opengl=..., --vulkan=..., at the time of the application build, or realm.

It can be installed in the realm:

```shell
user# ix mut --opengl=mesa/aco --vulkan=mesa/aco
```

Then all applications built in this realm using `ix mut`/`ix let` will use the selected driver.

Or for a standalone application:

```shell
user# ix build bin/gnome/text/editor --opengl=angle --vulkan=amd/vlk
```

How to get a list of available mesa drivers:

```shell
user# ix tool listall | grep mesa/ | grep -v mesa/dl | grep -v /fakes
lib/mesa/anv
lib/mesa/nouveau
lib/mesa/nvk
lib/mesa/iris
lib/mesa/soft
lib/mesa/aco
lib/mesa/radv
lib/mesa/radeonsi
```

Rule of thumb - if the name matches the name of the Vulkan driver from Mesa, then Zink will be chosen as the OpenGL driver - [https://docs.mesa3d.org/drivers/zink.html](https://docs.mesa3d.org/drivers/zink.html).

So, to use the Zink + Vulkan AMD RADV driver, run:

```shell
user# ix mut --opengl=mesa/radv --vulkan=mesa/radv
```

If zink + vulkan is a good choice for you, it is preferable because the ACO shader compiler is significantly smaller in size than the LLVM variant.

Also one can use:

```shell
user# ix build bin/gnome/text/editor --opengl=angle ...
```

For google's ANGLE opengl driver - https://github.com/google/angle

And:

```shell
user# ix build bin/gnome/text/editor --vulkan=amd/vlk ...
```

For AMD's https://github.com/GPUOpen-Drivers/AMDVLK

For regular applications we suggest use generic opengl driver, like ANGLE, and appropriate Vulkan driver:

```shell
user# ix build ... --opengl=angle --vulkan=amd/vlk # will use AMDVLK as Vulkan driver
user# ix build ... --opengl=angle --vulkan=amd/radv # will use Mesa RADV as Vulkan driver
user# ix build ... --opengl=angle --vulkan=intel/anv # will use Mesa ANV as Vulkan driver
```

For compositors, which rely on Mesa GBM for buffer allocation, we recommend using full Mesa stack:

```shell
user# ix build bin/niri --opengl=mesa/aco --vulkan=mesa/aco
```

Software rasterization:

```shell
user# ix build bin/quake/2/yamagi --opengl=angle --vulkan=swift/shader
user# ix build bin/quake/2/yamagi --opengl=mesa/soft --vulkan=mesa/soft
```

Also one can try to build packages with only one of --opengl/--vulkan provided, for opengl/vulkan-only experience.

## Oddities
* If you want to use Zink + Vulkan, it is recommended to add to your session script:
```shell
export WLR_RENDERER=vulkan # for wlroots-based composers
export MESA_LOADER_DRIVER_OVERRIDE=zink
```
* Intel cards work with --opengl=mesa/iris, but do not work with --opengl=mesa/anv. Also you can try --opengl=angle --vulkan=intel/anv.

<!-- {% endraw %} -->
