# Cases

> Why you should try **IX**

<!-- {% raw %} -->

**IX** obsoletes many ways of compiling statically linked binaries:

[Buildroot](https://buildroot.org/)

```shell
ix# ix build --target=linux-aarch64 --for_target=linux-riscv64 bin/gcc bin/binutils
```

[llvmbox](https://github.com/rsms/llvmbox)

```shell
ix# ix build --target=linux-aarch64 bin/clang/16
```

[BusyBox](https://www.busybox.net/downloads/binaries/)

```shell
ix# ix build --target=linux-aarch64 --purec=musl/pure bin/busybox/ix
ix# ix build --target=linux-aarch64 --purec=uclibc/ng bin/busybox/ix
```

[Static tmux](https://github.com/mjakob-gh/build-static-tmux)<br>
[Another static tmux](https://github.com/maciejjo/static-tmux)<br>
[and how to build it!](https://stackoverflow.com/questions/62620514/building-static-executable-tmux-on-linux)

```shell
ix# ix build bin/tmux
```

[How to build static Git](https://stackoverflow.com/questions/11570188/how-to-build-git-with-static-linking)<br>
[again](https://github.com/EXALAB/git-static)<br>
[and again...](https://gist.github.com/mishudark/3080857)

```shell
ix# ix build bin/git
```

<!-- {% endraw %} -->
