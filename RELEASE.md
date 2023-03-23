# Alpha Version of stal/IX: the first statically linked, source-based, bootstrapped rolling Linux distribution

<sup> March 23, 2023 </sup>

**We are pleased to officially announce the alpha release of stal/IX, which we believe is the first statically linked, source-based, bootstrapped rolling enterprise-grade Linux distribution with a Nix-like file system.**

Based on the **IX** package manager, **stal/IX** is an open source project committed to minimalism, simplicity, and security.<br>
**stal/IX** development to the current version took about two years.

Creator and lead developer **[Anton Samokhvalov](https://github.com/pg83)** about the project, *“**stal/IX** is an attempt to rethink some fundamentals without touching Linux API and ABI. One of the **stal/IX** goals - from the very beginning to build the system in such a way that it is possible to understand how it works, and not only use it conveniently”*. 

- - -

### Some highlights:

* Static linking for everything. There is no dynamic loader in the system, because all applications are linked statically. Even Gnome and the browser.

* New (without legacy), like nothing else, package base for 2000 packages. Thanks to the use of jinja templates, it allows you to separate the static build logic of packages from the logic of describing their dependencies and build settings.

* Nix/Guix-like file system. User level package management with ability to have different versions of the same packages for different users, and for the same user. Atomic distribution updates without ostree.

* By default, clang and libc++ are used to build programs, but, at the request of the user, it can be built with any other compiler. Moreover, it is possible to build the used set of programs with different compilers that are most suitable for this or that software.

* musl as system libc. And also - uclibc-ng, mlibc, and so on.

* runit to manage system processes and to manage user session.

* sndiod for audio playback.

* Wayland only, no tearing, stuttering, and no X.

* And also - it is built literally "from nothing", except for the source code and the compiler!

- - -

Unlike traditional Linux distributions that rely on dynamic linking and shared libraries, **stal/IX** is statically linked.

Our source-based approach allows users to build their own packages from scratch, giving them complete control over their system, while our rolling release model ensures that users always have access to the latest updates.

One of the most important features of **IX** package manager - its design, which allows users to use it both as a basic package manager in **stal/IX** distribution, and separately, in any supported OS (Linux, macOS).

We believe that our innovative approach to package management and distribution will provide users with a faster, more secure development experience and build simplicity. 

The alpha version of **stal/IX** is available for early adopters to try out and provide feedback. Contributors are welcome!

- - -

**[Download](https://github.com/stal-ix/ix)** the alpha version of **stal/IX**.<br>
To learn more about **stal/IX**, visit our website at [https://stal-ix.github.io](https://stal-ix.github.io).

- - -

P.S. And this is how [AI writes](RELEASE_AI.md) about us:)
