# stal/IX blog

> About building systems in a different way, static linking, and much more!<br>
> And about the stal/IX, of course, from the benevolent dictator [(@pg83)](https://github.com/pg83).

- - -

## One more why for stal/IX
<sup> January 11, 2023 </sup>

#ix #stal/ix #loginshell #interactiveshell

In previous posts I wrote why I started my Linux distribution (Why stal/IX. [Part 1](BLOG.md#why-stalix-part-1), [Part 2](BLOG.md#part-two-why-this-way-not-otherwise), [Part 3](BLOG.md#why-stalix-part-3)).

In short, a lot of things done in Unix/Linux piss me off, and the easiest way to fix them is to have control over the whole environment!

Recently I was figuring out with a colleague why ssh works weirdly on a machine with stal/IX installed. The "weirdness" was the difference in behavior of ssh daemon in interactive vs. batch mode.

In fact, this weirdness is very easy to explain - it is caused by a confusion between the login shell and the interactive shell!

Unfortunately, these modes often coexist, so there is a temptation to stuff your settings for the interactive shell into the configuration files for the login shell.

But this is fundamentally wrong!

Login shell is, if very roughly, the first shell in the chain for your user. Usually some display manager starts it, like mingetty, agetty, emptty, greetd, sddm, and thousands of them; ssh, putty, telnet, mosh, and so on, are also things that spawn the first shell in the chain!

The interactive shell is where you enter commands from the keyboard.

These are, of course, quite different things.

(By the way, a little aside - if you have cases when the same elements are repeated in PATH - this means that you do not understand the difference between these entities, and sometimes the construction of adding something to PATH works twice, because it's in the wrong file.)

The problem here is further exacerbated by the fact that different shell read different startup files, depending on the mode.

For example, bash reads /etc/profile in both interactive and login mode. And most other shell don't!

In short, trash, waste, and sodomy, layering of habits, misunderstandings, and so on are clearly visible here. Evaluate for example only /etc/environment - [https://superuser.com/questions/664169/what-is-the-difference-between-etc-environment-and-etc-profile](https://superuser.com/questions/664169/what-is-the-difference-between-etc-environment-and-etc-profile) - this is upyachka, born from the fact that sometimes you need to do "login", but without "shell", but variables have to be taken from somewhere!

So, of course, my intention in stal/IX is to do everything "right" **(\*)**, and not to reproduce the hack that everyone is used to!

Why?

Because if this hack suited me, I would install Fedora! And I need better!

**(\*)** Right - of course ssh and others should start the first shell in login mode. Even better - for each shell, the user must figure out which files to put interactive settings (such as completion), and which - login.

- - -

## Non-boring routine updates
<sup> January 01, 2023 </sup>

#pkgconfig #CI

Handled routine software updates.

CI is very cool and convenient, because how else would I notice that, with the new libpcap, the wireshark build started to fall?

It started crashing with a very strange error message:

```shell
"ninja: Entering directory `/ix/build/mcpe98eHnOfPdF2O/obj'
ninja: error: 
'/ix/store/iDo8RMwHvMz5h6hQ-lib-pcap/lib/libpcap.a;/ix/store/3KBy5KQKrWQR6EuF-lib-nl/lib/libnl-genl-3.a;/ix/store/
3KBy5KQKrWQR6EuF-lib-nl/lib/libnl-3.a', needed by 'run/tshark', 
missing and no known rule to make it"
```

It really says here that cmake generated an incorrect ninja file - somewhere there, in the list of dependencies of some target, has got such a strange line - 3 static libraries combined through `;`.

In fact, it's pretty clear that cmake just writes there some line that it received from pkg-config, and something unexpected turned out to be there.

Here is a diff in libpcap.pc of two different library versions:

```shell
Name: libpcap
 Description: Platform-independent network 
    traffic capture library
+Version: 1.10.2
+Requires.private: libnl-genl-3.0 
+Libs: -L${libdir} -Wl,-rpath,${libdir} -lpcap
-Version: 1.10.1
-Libs: -L${libdir} -lpcap
```

Suspicion, of course, caused the Requires.private field [https://people.freedesktop.org/~dbn/pkg-config-guide.html](https://people.freedesktop.org/~dbn/pkg-config-guide.html).

*"Requires.private: A list of private packages required by this package but not exposed to applications. The version specific rules from the Requires field also apply here".*

The most confusing explanation, which IMHO does not explain anything.

I understand this as the beginnings of package management in pkg-config - a set of libraries that should be in the system (because they are needed by dependencies for some .so  in the package).

In the case of static libraries, we get just such a strange hat, from all absolute paths, combined through `;`.

What incorrectly called or understood what, I did not begin to understand, because I already have information about transitive dependencies, no need to duplicate it in .pc files.

So I just took down it to fuck from all .pc - [https://github.com/pg83/ix/commit/bac7f907bc4d8841e4eff9aba93c2a0bd765fc96](https://github.com/pg83/ix/commit/bac7f907bc4d8841e4eff9aba93c2a0bd765fc96).

In general, I try not to use such global text replacements for generated files, but sometimes you can’t do without it, otherwise static build support would turn into huge patches on top of all known build systems.

For example, here is a fix that needs to be applied to all meson.build files - [https://github.com/pg83/ix/blob/main/pkgs/die/c/meson.sh#L107-L109](https://github.com/pg83/ix/blob/main/pkgs/die/c/meson.sh#L107-L109).

Unfortunately, in meson (and in cmake, but not in autoconf!) the author of the build scripts can say "build this as .so, even if the user of the package asked to build statically" without the possibility of override.

- - -

## Generating stable unique keys
<sup> December 21, 2022 </sup>

#bootstrap #ix

I solved a long-standing problem here - generating unique keys for various kinds of programs, but so that they are stable from generation to generation, that is, if you need to rebuild the package with the same input data, then the resulting keys should be the same.

Usage example:
```shell
# ix mut system bin/dropbear/runit --seed="dead beef"
```

When installing dropbear (this is such an alternative to ssh) it wants to generate host keys that should not change for about the entire lifetime of the installed system.

It uses random() for this, which does not correspond to the bright idea of "clean build" - the package is a pure function from its inputs (I remind you that this is an important condition for the ability to reuse RO packages between different users (not necessarily even on the same host)).

Actually, the solution to this problem is divided into 2 parts:

* Slightly rewrite the key generation utility so that it does not go to /dev/random, but to a pre-prepared file - [https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/keygen/ix.sh#L7](https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/keygen/ix.sh#L7).

* Submit this file to the host key generator and generate these keys. So we get a "clean" package, which depends only on its seed parameter - [https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/hostkeys/ix.sh#L14](https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/hostkeys/ix.sh#L14).

Thus, we have, as it were, a "random" set of keys, which is a pure function of seed. The task of the user of the system when setting up the machine is to specify this same seed so that it is unique (this can be done by the installer transparently for the user).

In the process, I got an interesting artifact - a "package with prepared entropy". This is how I ask that it be available at the time of preparing the keys - [https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/hostkeys/ix.sh#L4](https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/hostkeys/ix.sh#L4), and this is how its implementation looks [https://github.com/pg83/ix/blob/main/pkgs/aux/entropy/ix.sh](https://github.com/pg83/ix/blob/main/pkgs/aux/entropy/ix.sh),
[https://github.com/pg83/ix/blob/main/pkgs/aux/entropy/ix.sh#L10](https://github.com/pg83/ix/blob/main/pkgs/aux/entropy/ix.sh#L10) - here is all the pulp, we pass seed to openssl.

```
pg-> ./ix build aux/entropy 
  --entropy_seed="dead beef" 
  --entropy_size=100
READY /ix/store/7aP.../touch
pg-> ls -ls /ix/store/7aP.../share/entropy
     100 ... /ix/store/7aP.../share/entropy
```

The artifact is actually quite useful, because at system setup time there are quite a few processes that require a unique host ID to be generated, such as /etc/machine-id, which is used by dbus.

In general, it is clear that we do not worsen the process compared to how it was before - it was "we use the entropy at the current time as a seed to generate some files", it became - "we save the entropy at some point in time, and use it to generate the same files over and over." That is, we store not the result of the generating, but its seed.

By the way, I found beautiful while reading the source - [https://github.com/mkj/dropbear/blob/master/dbrandom.c#L270](https://github.com/mkj/dropbear/blob/master/dbrandom.c#L270).<br>
Here it is written how normal people build their initial entropy for work.

(Those who tried to blame me for the file [https://github.com/catboost/catboost/blob/master/util/random/entropy.cpp#L40](https://github.com/catboost/catboost/blob/master/util/random/entropy.cpp#L40) should be ashamed - everyone does it!).

- - -

## Ranting on glibc
<sup> December 16, 2022 </sup>

#glibc #rant

**Friday evening rant**

There is such a library, #glibc, whose authors are known for wanting to do things "better" than everyone else.<br>
The problem is that, more often than not, their solutions turn out to be just "different", and then - "fuck, who, and what a fuck, invented it all".

There is such a function - pthread_cancel().

Do not use it anywhere and ever. Its behavior is as poorly defined as possible, and, in real life, it will lead you into some kind of deadlock. Because you won’t write tests for this section of code anyway.

By standard, all it has to do is call the specially registered cleanup handlers and end the thread. But colleagues from glibc decided that this was not enough for them! And they made the next feature - they call stack unwinder to destroy local objects on the stack in this thread.

How is it done?

* We load in runtime unwinder from gcc. No, it's not necessarily the same unwinder used by the rest of the program, it can be statically linked. [https://codebrowser.dev/glibc/glibc/nptl/pthread_cancel.c.html#99](https://codebrowser.dev/glibc/glibc/nptl/pthread_cancel.c.html#99)

* We throw an exception unknown to science. [https://codebrowser.dev/glibc/glibc/nptl/unwind.c.html#130](https://codebrowser.dev/glibc/glibc/nptl/unwind.c.html#130)<br>
Question for experts  - how does this exception interact with C++ exceptions? What about catch (...) ?

* I did not find the code that catches it on the top of the stack, but before it was also very intricate. Normal people would make one source with a .cxx extension, and catch it through the normal exception mechanism, but that's not the glibc developers way.

In fact, if your program brings its own unwind runtime with it, expect an arbitrary crash as a result of interacting with glibc/pthread_cancel.

- - -

## Tired with ~~all these~~ temporary files
<sup> November 21, 2022 </sup>

#temporaryfiles #stal/ix

Today I have an anecdote for you about my sense of beauty.

I personally get pissed off by programs that want to create temporary files. In general, a "temporary file" is some kind of nonsense, because what a fuck to write ephemeral data to a persistent file?

Data can either be buffered in memory, or written through a pipe to the input of another program, or something equally reasonable.

Usually programs write something to temporary files or in the form of a hack, when some block of code already accepts int fd, and rewriting it to a normal interface is too lazy, or when some program in the chain does not know how to pipe.

In the first case, by the way, in modern linux you can use memfd_create (), a pretty useful thing in terms of "sweep the trash under the bed."

Another problem with temporary files is that all sorts of strange programs do not respect the TMPDIR setting, and try to write directly to /tmp, moreover, coming up with some wild ways to differentiate access and "not intersect" with other instances.

Here is the content of my session tmp:

```
./epiphany-pg-aaNPpb
./d9ca075b14fe84b587843f702e0d2466-
     {87A94AB0-E370-4cde-98D3-ACC110C5967D}
./6b047d326310867001cb39f5218a36ba-
     {87A94AB0-E370-4cde-98D3-ACC110C5967D}
./mc-pg
./mc-pg/mc-pg
./sway-ipc.10000.9709.sock
./wayland-1
./wayland-1.lock
./dbus.sock
./dbus-1
./dbus-1/services
./dbus.cfg
./ssh-XXXXXXJIAMlN
./ssh-XXXXXXJIAMlN/agent.9704
```

How many different ways of “uniqueness” did you count here?

I repeat, this infuriates me wildly, and I set myself a goal - all the programs that I use should be able in TMPDIR.

By the way, a little aside, TMPDIR, of course, should be a "complex" path that includes the user id and the session id (for example, to be able to effectively clean up this junk at the end of the session):

```
pg->echo ${TMPDIR}
/var/tmp/10000/10953
```

That’s why I don’t have a /tmp root folder in stal/IX.

Why?

Because, by a strange coincidence, all programs that have their own strong opinion about the naming of the folder with temporary files, try to write to /tmp.<br>
And I don’t have it, but I do have /var/tmp with separation by sessions.

Therefore, all such programs break, and I fix them all.<br>
To make this process not painful, I created a small library for this - [https://github.com/stal-ix/ix/tree/main/pkgs/lib/shim/ix](https://github.com/stal-ix/ix/tree/main/pkgs/lib/shim/ix).<br>
Its application is almost automated - [https://github.com/stal-ix/ix/blob/main/pkgs/bin/got/ix/ix.sh#L10](https://github.com/stal-ix/ix/blob/main/pkgs/bin/got/ix/ix.sh#L10).

- - -

## Static VS dynamic linking
<sup> November 02, 2022 </sup>

#staticlinking

In the "static vs dynamic linking" battle, there is one factor that no one really talks about, or I just haven't met before.

This is aesthetics!

Moreover, it’s not simple aesthetics, that, they say, in the case of static linking, all sorts of different .so are not lying around the entire fs, about which, without a package manager, it’s impossible to say where they come from and what they are for. It lies on the surface, and is not very interesting.

Once, when I was talking about #mesa, I casually mentioned how it is building.

It was about the fact that, in the process of building 3 .so’s, 10 .a’s are built, and they are somehow, rather arbitrary, combined into the corresponding .so files, using a linker script to hide common parts.

Why is this being done? Why not make 10 .so’s that just link to each other like they are supposed to, and there is no code duplication magic and all that?

This is what dynamic build aesthetics demand!

Because it’s not good to dump a bunch of incomprehensible .so on the user, without some kind of understandable scope, which do who knows what. It is necessary to give the user a clear set of artifacts, with a clear scope, but the fact that the code is duplicated between them - who cares?

By the way, my recent story about the duplication of symbols in the #gtk and #wayland libraries can be attributed exactly to the same topic. It's crazy to separate these 3 unfortunate functions into some completely artificial library, how to sell it to the user, how to package it in distributions, and other unpleasant questions.

Conversely, static linking encourages more and more granular dependencies, because these are purely build-time artifacts and are not visible to the user. Therefore, they can be arbitrarily shitty, at least 1 function in each, if it is so convenient, no one will see it live anyway.

*(an example of such a division of libraries into parts - in the next series, and here is already a lot of text).*

So the inherent aesthetics of a particular linking method actually dictate how the code will be split into modules, and (my personal opinion) static build leads to a better, reflective the code essence, separation.

- - -

## Why stal/IX. Part 3
<sup> May 31, 2022 </sup>

#stal/IX #IX #packagemanagement #packagemanager

Prereq: [Part 1](BLOG.md#Why-stal/IX.-Part-1), [Part 2](BLOG.md#Why-stal/IX.-Part-2)

Many times I promised to write why my Linux distribution, and why it is designed this way.

### Part three. About the package manager design

By and large, I had 3 iterations to the package manager design.

**First iteration**

Packages were a description in Python, the graph executor for each node launched Python, which executed this code. Homebrew is about the same, only on Ruby.

Very quickly, I realized that Python is completely unsuitable for this task:

- It is absolutely not composable, you cannot concatenate 2 scripts and get a working script, due to the fact that alignment and indentation are important in Python. Therefore, the task "take this script and replace the build phase with this one" was completely inoperative.

- The second reason is common for both ruby and python, and for everything except posix sh. In these languages, you have to invent some dsl to describe simple actions, that are done in the shell with simple commands:

```
os.setcwd('xxx')
a = os.environ['yyy'] + '/zzz'
os.setcwd(a)
```

vs

```
cd xxx
cd ${yyy}/zzz
```

In short, this DSL looked shitty, and I definitely didn't want to give it to the user.

**Second iteration**

Build scripts in Arch style, with meta information in the comments:

```
# url https://a.b/c.tgz
# dep lib/c lib/c++
# bld bin/make

build() {
    make -j 8
    ...
}

...
```

This was already better, but:

- The task "take this build script and change a little build() in it or list of dependencies" was still not solved in a reasonable way, so the bootstrap chain contained a lot of redundant code. Well, like, I had to copy the build description for building cmake at the earliest stages, when there is nothing yet, or already for later stages, when the previous cmake and libraries are available.

- Here I was already beginning to strain that I had to repeat the same boilerplate to call meson/cmake/etc. I must say that the same Arch can call cmake directly. And I need to pass 100500 parameters in order to provide a static build, and nix-style paths in the file system (in short, DESTDIR and PREFIX are not the default).

**Third iteration**

At this point, I had jinja - a language for describing and substituting templates, and the packages began to look like they look now.

[https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/kitty](https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/kitty) - look, t/ contains a template that specializes a common template for some build system, and its two heirs - linux/, darwin/. Check them out before reading the following paragraphs.

It is important to understand here that not a shell script is built from this template, but json, which contains pieces of shell scripts for different phases (build, configure, etc) of the build, and lists of dependencies.

- The tasks "do me the same thing, but with mother-of-pearl buttons" are elegantly solved - expanding and narrowing the list of dependencies, replacing and arbitrary modifications of different blocks.

- It becomes possible to pack all the logic associated with a particular build system into a template, and make it more and more complex.

*Examples:*

* [https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/brotli/ix.sh#L7](https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/brotli/ix.sh#L7) - and build me brotli, but with an extended list of libraries, because I collect not a lib, but binaries;
* [https://github.com/stal-ix/ix/blob/main/pkgs/bin/curl/ms/ix.sh](https://github.com/stal-ix/ix/blob/main/pkgs/bin/curl/ms/ix.sh) - build curl for me, but with a different http3 library, and fresher sources;
* [https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/gdbm/ix.sh](https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/gdbm/ix.sh) - build me a binary from libgdbm and add readline to it (pay attention, we are expanding not only a list of libraries, but also configure flags);
* [https://github.com/pg83/ix/blob/main/pkgs/die/c/cmake.sh#L43](https://github.com/pg83/ix/blob/main/pkgs/die/c/cmake.sh#L43) - base template for cmake, see how much squatting I do there, from passing paths to the compiler, before replacing SHARED with STATIC in CMakeLists.txt - [https://github.com/pg83/ix/blob/main/pkgs/die/c/cmake.sh#L81](https://github.com/pg83/ix/blob/main/pkgs/die/c/cmake.sh#L81).
Thanks to this, my build files that the user sees are lean and mean, there is only information on the essence of things (dependencies), and all these squats are hidden from the user.

All things are difficult before they are easy, at some point in time it became clear to me that the template engine very easily solves the problem of the build' arbitrary setup of one or another target.

- - -

## Why stal/IX. Part 2
<sup> May 29, 2022 </sup>

#stal/IX #filesystem #staticlinking

Prereq: [Part 1](BLOG.md#Why-stal/IX.-Part-1)

Many times I promised to write why my Linux distribution, and why it is designed this way.

### Part two. Why this way, not otherwise

**Directory structure**

Everything is quite simple here, I don’t understand how nowadays one can make a new distribution according to the LSB/FHS [https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard).

This is some kind of terrible legacy left over from the times of the Unix beginning, with a bunch of poorly solved problems (dll hell, atomic updates (ostree is a hack, and don't try to convince me otherwise)).

FS should be a content addressable storage, like git/nix/guix.

So we get almost free, just to name a few:
- Multiversioning;
- Fast rollovers and rollbacks;
- Atomic updates (switching the entire configuration - switching 1 symlink);
- Free caching of build artifacts in the batch system;
- Non-root/user package management. This is a direct consequence of content adressable storage.

**Functional configuration management**

The point is that, in fact, the contents of etc/ are determined by a number of #realm settings, and a description of the package base.<br>
You can’t go into etc/ with your hands, it is generated by the #realm preparation scripts, and in general, read-only.

I won’t write much here, who knows, he understands, and I still won’t explain to the rest in 2 sentences.

**Static linking**

A good text with a summary, why the main complaints about static linking do not stand up to serious criticism - [https://gavinhoward.com/2021/10/static-linking-considered-harmful-considered-harmful/](https://gavinhoward.com/2021/10/static-linking-considered-harmful-considered-harmful/).

I think you already understood that I consider Linus still that ghoul, but the enemy of my enemy is my friend - [https://lore.kernel.org/lkml/CAHk-=whs8QZf3YnifdLv57+FhBi5_WeNTG1B-suOES=RcUSmQg@mail.gmail.com/](https://lore.kernel.org/lkml/CAHk-=whs8QZf3YnifdLv57+FhBi5_WeNTG1B-suOES=RcUSmQg@mail.gmail.com/).

- Tooling for static linking is still better because it's as simple as ABC;

- Judging by the forums, in Linux there is an eternal problem with dynamically loaded plugins - something was not found at runtime, and because of this nothing works. I have linked - it means that the declared functionality will be available;

- The resulting binaries, despite some duplication of the object code, are faster;

- FS is cleaner, there is no unnecessary garbage, I can tell about each file what it does. In nix/guix, for example, there can be dozens of variants of the same library, this is ungrepable noodles;

- As a consequence, it is aesthetically easier to have different versions of the same library, because they do not annoy the eyes;

- Personal factor. For the last 15 years I’ve been implanting static linking in Yandex, and, in general, I have succeeded in this. I have the ambition to show the world that based on this model, you can get a full-fledged, not a toy, distribution. Toy ones, such as stali, oasis, and so on - this, excuse me, is a school craft against the background of what I have already done.

**Healthy minimalism. The possibility to view the entire system**

No code bloat, no systemd, no pipewire(?), etc.<br>
Overall, no vendor lock on RedHat/IBM software.

The Red Hats really want to lock down the Linux ecosystem on themselves. On systemd, on gstreamer, as part of pipewire, etc.<br>
I don’t see anything good in this, all this software interferes the ability to understand how the system works.

I’ll just briefly note here that I don’t mind if someone makes a variant stal/IX on top of systemd, I don’t interfere with this, and I’m even ready to help.

Look through this [folder](https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/dbus), even runit scripts are not part of the main package, so that you can have systemd/s6/etc units in parallel.

- - -

## Why stal/IX. Part 1
<sup> May 28, 2022 </sup>

#stal/IX #OS

Many times I promised to write why my Linux distribution, and why it is designed this way.

### Part one. What for

There are a lot of things I don't like about modern OS:
* I don't like the Linux scheduler;
* I don't like private parts in macOS, its policy towards applications and, especially, developer tools (profiling, debugging, etc. - these all stop working “out of the box”);
* For example, I really don't like that in the Unix orphan process is nailed to init, I want all my process runnings not to double fork to see a completely supervised tree;
* I don't like the complexity that RedHat brings to Linux, for example, systemd is a general-purpose (and very bad!) dynamic task graph launcher, the boot management system is there on the side (correct, by the way, done in macOS, there is a separate bootloader, a separate socket activation, and so on). PipeWire - a universal handler for the multimedia stream graph, and fuck no one needs it in such a quality, all sane (yes-yes, "no true Scotsman") players take oak ffmpeg, without this shitty dynamics and resolving plug-in dependencies at runtime;
* I don't like the layering of shit on crap in modern LFS/LSB distributions. Ah, dynamic linking leads to dll hell, well, we'll leave it, but attach ostree and containers for deploying applications. You know, this is a face-saving decision, not a rethink, as it would be right to do from the very beginning.

This list is endless, but I think you get the point.

I've been thinking for a while about how, with the least amount of effort, to solve the most problems that annoy me.

Write my own OS? I'm trolled with this about once a month, and, by the way, I would make a good OS (somehow I need to write how it would be designed). The only problem is that I will launch a modern (currently) graphical application there in 15 years, and then, if I don’t get too fucked up with architecture and tools.

I imagine this quite accurately, you can just look at the dynamics of Linux development, and extrapolate it based on:
* take a normal language to write 10 times faster than Linux developers do;
* take the normal architectural principles of building the kernel.

Figures about the kernel size [https://www.opennet.ru/opennews/art.shtml?num=57260](https://www.opennet.ru/opennews/art.shtml?num=57260).<br>
I roughly understand that Linux became ok somewhere around 2.6, 5 - 10 MLOC. If we take a normal language, we need to code somewhere 0.5 - 1 MLOC.<br>
In general, everything is predictable, and very sad.

In the end, I decided that most of my problems can be solved by a normal Linux distribution, which I understand well, and can "bend" in the most appropriate way.<br>
For example, having created a demon, which smites all those nailed to the init orphan process, gradually and incrementally.<br>
For example, refusing dynamic linking and dll hell as a CLASS of problems.<br>
How to make it in the same Fedora - you can kill yourself.

About the overall complexity of the system, and how I'm trying to struggle with it - well, about this, in fact, there is my entire blog.

From all of the above it follows:
* it's completely pointless to ask me "why not systemd"

* "why not ostree"

* "Why ...

I'm trying to solve the problems known to me in a different way, this is a research hobby-distro, and not a competitor for RHEL in its field.

If I'm satisfied with systemd and dll hell, I'll take Fedora and won't bother.

But I'm curious about "how, in general, it is possible otherwise."

- - -

## Mesa, light of my life
<sup> March 23, 2022 </sup>

#mesa #meson

Mesa - light of my life, sparks at my fingertips. My sin, my love. Me-sa.

To build it, the now very popular build system meson was developed, and, in fact, this is very sad, because Mesa developers know meson too well, and use any of its fucking features.

In fact, Mesa consists of 2 parts - a plugin loader, which, on top of the plugin interface, implements all kinds of state trackers like opengl, vulkan, and, actually, plugins.

Since the Mesa authors know meson too well, the Mesa build looks something like this:
* K object files are compiling;
* N archives are compiled from them, and, what is important, there are intersections - the same .o can end up in several archives;
* From these K + N artifacts, M final .so are compiled. Also with arbitrary intersection in .a/.o files!

It turns out that each final artifact contains an arbitrary subset of sources.

As long as it’s all built into .so, with hidden characters, this isn't a problem other than increasing the amount of final code and compile time (because some sources are still rebuilt several times).

If we link statically, then we get understandable problems - repeated characters when linking.

When I first compiled Mesa, I solved it in a very strange way - I dumped all the end artifacts into one trough, and made one mega-library out of it, which replaced all the end artifacts.

Overall, it works well, except for one very important scenario - I still need to cut this giant artifact into 2 parts - a loader and a driver, and make 2 physically different dependencies out of them.

This is necessary in order to raise the dependency on the drivers to the final programs, and all intermediate libraries should depend only on the loader. This is an important requirement if we don't want to rebuild the entire tree of libraries needed to build the final GUI programs for each set of drivers, and, in fact, for the normal binary build cache.

I approached the task of splitting 5 times:
* Split to .o files the final artifact manually. Breaks down from a combinatorial explosion of various ways to compile drivers;
* Make it the way that everything is compiled into several completely independent .a files - everything rested on the defects of the meson build system and the features of the mesa build files - here I tried to manually edit the Mesa build scripts (does not scale for updates), and meson itself (here I was closest to victory, but did not work out).

In short, so far I solved this problem like this:
* Wrote a generic procedure for subtracting one .a from another. This cannot be done by the names of .o files, for the reasons that I have already described above, so everything is done character by character [https://github.com/pg83/ix/blob/main/pkgs/bld/librarian/substr.py](https://github.com/pg83/ix/blob/main/pkgs/bld/librarian/substr.py).
* By sequential subtraction puted the bootloader and driver library in order. 

This is much more aesthetically pleasing than the previous .o sack idea, and works quite well (at least it solves the original problem).

- - -

## About build systems
<sup> March 09, 2022 </sup>

#OSS #buildsystems 

I am saddened by the state of modern OSS build systems. Today I'll tell you about such an aspect: every self-respecting modern build system wants to have a package manager in itself.<br>
That is, to provide not only the execution of the build graph of one project, but also of all build graphs of all dependencies.

Have you seen cargo? I wrote a couple of times what this claim for comprehensiveness leads to in relation to cargo - the need to wrap all non-cargo dependencies in the cargo build. This looks ugly, and leads to problems with diamond-driven dependencies.

The problem is that, despite all the attempts of these build systems authors, they do not become comprehensive, and it is impossible to live within the same ecosystem. Therefore, each such system wraps all external dependencies into itself. This, excuse me, is the square (from the number of build systems) in terms of the complexity of the efforts made.

I already wrote about .wrap files from meson (there is a whole repository for them - [https://mesonbuild.com/Wrapdb-projects.html](https://mesonbuild.com/Wrapdb-projects.html)).

This is something to write about endlessly, here are some very shitty examples:
* nodejs rewrites the build system from v8 to autoconf;
* webkit remakes the build system from ANGLE (Google's implementation of opengl) to CMake;
* chrome vendors a bunch of libraries, I won't describe them individually;
* telegram vendors all its dependencies and builds them in a way that is not compatible with anything.

Fun fact - build systems make life easier for developers, but completely kill the maintainers of this software in the repositories, because de-vendoring all this shit is painful.

I live with this a little easier than any Fedora there. In the case of dynamic linking, vendoring is also path crossing in fs. In the case of static linking, this is at least not visible to the outside, it is enough to de-vendor all sorts of freetype/fontconfig and so on.

Chrome, by the way, is great in this regard, they help de-vendor those parts that are simply necessary (such as font rendering).

Among other reasons, it's painful, because every OSS project, from small to large, invents its own way of vendoring.

Ideally, it would be very cool if the build systems stopped doing such garbage, and agreed with the package systems on the interaction interface. It's actually not very difficult.

Imagine the command:

```shell
mix run lib/z lib/freetype bin/make
     bin/cmake bin/clang/14 bin/ninja -- make -j 16
```

This command will make #realm, in which the specified libraries, specified build tools, and (here it is important!) wrappers for the cc/c++/cpp compiler (well, or rustc, whatever) will be available, which will automatically set up the necessary paths to libraries and headers files.

Seems complicated? Well let's simplify it into alias `mixrun=mix run $(cat mix.shell)` - and use it like this:

`mixrun make -j 16` or `mixrun --sanitize=address --opt=-O2 make -j 8`.

Then in the corresponding makefile you don’t need to do autodetect at all, list all sorts of -I / -l-L / etc, but just call simple commands like
`cc -c x.o x.c`.

The same works for cargo, and for any other build system.

The main point is that the project-level build system should not autodetect the presence of dependencies and deliver them. Nix can do that, IX can do that.

The problem is that, at any given time, it is easier for the author of this or that OSS project to crutch up the next vendoring, instead of going to negotiate with all interested parties.

Separately, I note that these kludge package managers are completely crap. I would really like to see how cargo, for example, tries to vendor any lib with settings and data for this lib.

- - -

## The security model. Episode 2
<sup> March 08, 2022 </sup>

#security

Continuing the security model theme:

[https://www.opennet.ru/opennews/art.shtml?num=56818](https://www.opennet.ru/opennews/art.shtml?num=56818)<br>
[https://www.opennet.ru/opennews/art.shtml?num=56812](https://www.opennet.ru/opennews/art.shtml?num=56812)

Two more vulnerabilities that I consider unimportant. And some considerations about their implementation:
* All sorts of complex security models are badly "linked", and they turn out to be somehow badly interacting. Like selinux + cgroups.
* The more complex the system, the more difficult it is for a leatherbag to think about it.

Therefore, I believe that models like "everything is permitted / everything is prohibited" rule because of their simplicity - it is more difficult to mess up in their implementation, and it is easier to think about them. And, despite the fact that they cannot express all sorts of shitty complex policies, they turn out to be safer.

How to deal with inherent complexity? I believe that with the help of a hierarchy, the top level is divided into two "all / nothing" layers, within each layer you can run a container with the same simple division.

- - -

## About the security model
<sup> February 25, 2022 </sup>

#security

I promised to write about the security model here.

I’ll make a reservation right away, I’m writing about the security model of a personal laptop or desktop computer, the reasoning below does not apply to servers or even to your phone. It is also not applicable for any kind of corporate and BYOD laptops.
* It is convenient to consider this model as a two-user one: root + user, the fact that some system processes are running by nobody, actually doesn't change anything.
* All valuable data on your laptop (credit cards, passwords, access keys etc.) belongs to your personal account, root owns only system files and daemons.
* Therefore, if malware gains access to your user, then it gains access to all useful content on your computer. Getting root rights is NOT the target of the attack. Root on such a system is not needed for anything, not even needed to run a regular background process, because in any modern OS there are session user process supervisors. Let's say dbus.
* Hide your presence - is also not about root, BTW, these are slightly different holes in the system.

Hence an interesting consequence:

It is convenient to think of root not as a user who can do anything, but as a way to separate dangerous operations that require confirmation and all others. That is, sudo in such a system is not an escalation of privileges, but a signature that you run the current command, understanding what it does. Well, and, accordingly, a guarantee that the usual rm -rf $STEAM_ROOT/ will not demolish the entire root [https://github.com/ValveSoftware/steam-for-linux/issues/3671](https://github.com/ValveSoftware/steam-for-linux/issues/3671) (yes, I know that now rm - rf / works a little differently).

Therefore, I have a rather disregard for all sorts of CVE for #system0. It's cool, of course, that there is some kind of passage that allows attackers to get the root locally, but I hope that I was able to explain that sometimes it’s neither hot nor cold from this.

A separate issue - where the security perimeter in your laptop lies then. The answer is quite obvious - it lies in your browser. You download software from stores, you don’t run any shit from the Internet. The perimeter is in the sandboxing of the code running in the browser. In the processing of images by the browser. And so on. The browser must be loved, groomed, updated. In my opinion, the browser is the only (well, okay, sshd also) program in the system that makes sense to lick from a security point of view - all sorts of libc hardening, ASLR (pic), etc.

And certainly not in your /etc.

Why is this not about the phone? Because your phone is not a two-user environment, but a multi-user one, the users don't trust each other.

Why not about BYOD? Because in BYOD there are 3 users: you, root, and comrade major (who needs to both hide his presence for malware and detect it). Also an interesting model, but some other time.

A two-user computer with antivirus, by the way, is also a slightly different model. I do not use antivirus, I consider it a thing, on average (the medicine is worse than the disease), harmful.

- - -

## Suffering by icons
<sup> February 04, 2022 </sup>

#svg #icons #rendering

With icons I suffered, of course:

* Paths to default icons in toolkits;
* What icons should be;
* Some of the icons in Adwaita are in png, some are in svg, these sets partially intersect. What this or that application will choose, one (idk who) knows. B&W icons are in svg, colored icons are in png;
* Svg loading is arranged through a plugin for gdk-pixbuf. Plugins for it are arranged in the most unnatural way of all known to me (I will try to write about how perversely developers arrange the loading of their plugins). Unfortunately, the latest version of this plugin, not rewritten in Rust, renders Adwaita's latest icons not very correctly;
* Debugging "what's actually loading" with strace doesn't really help, because every icon folder needs to contain an index.

That’s how we live - for high-quality rendering in png inkscape is required, for low-quality on the fly - Rust.<br>
Unfortunately, the Rust compiler doesn't even run right now, because it basically can't be linked statically.

All hopes for [#mrustc](https://github.com/thepowersgang/mrustc). Dude is getting closer and closer to supporting 1.54. It is clear that without a borrow checker, but who needs it not in the development process? It has its own implementation of proc macro (for obvious reasons), which does not require loading .so into the compiler.

I have now settled on svg icons with not quite correct rendering, because I've already tired of licking Epiphany ([https://wiki.gnome.org/Apps/Web](https://wiki.gnome.org/Apps/Web)). We'll have to see what's in store for the Chromium build path.

By the way, I almost won tearing in Epiphany. To do this, I built a webkit web process (which deals directly with rendering) on gtk4, in which everything is better with support for opengl / vulkan canvas, and the browser itself with gtk3. Because Igalia has already ported WebKIT to gkt4, but the browser itself has not yet. I think Igalia would choke on something for joy, knowing that I do this.

- - -

## About System0
<sup> January 31, 2022 </sup>

#system0

I think you already understood that #system0 is "my beauty":)

In it, I intend, to the best of my ability, to correct various problems that I encountered while interacted with Linux/Unix.

One such problem is the Unix property that the init inherits processes whose parent has died. I think that this is trash, waste, and sodomy (and one of the reasons for the emergence of cgroups, by the way). I'm pleased to see a completely supervised tree, if you know what I mean. This is convenient for both understanding and debugging. Long-lived processes need to be started using the session supervisor’s spawn command.

Therefore, I have [https://github.com/stal-ix/ix/blob/main/pkgs/bin/sched/staleprocs/staleprocs.sh](https://github.com/stal-ix/ix/blob/main/pkgs/bin/sched/staleprocs/staleprocs.sh) - a daemon that kills "stray" processes. If it flies, then I, of course, will add it to my init.

By the way, killing via SIGINT is there solely because if sway is killed via SIGKILL, then it regularly hangs the system tightly. Here is such a cool graphics stack in Linux.

Unfortunately, the same sway for some reason starts all processes through double fork.<br> 
Luckily, the fix is 3 lines of code.

- - -

## IX package manager: privilege escalation and some other implementation aspects
<sup> January 25, 2022 </sup>

#packagemanager

I been reading PulseAudio sources. I realized that Pottering has been writing the same program all his life - a graph for processing some thread of events in real time, and it is imperative that the graph be connected at runtime, and without checking that this graph can be executed in real life. Well, this idea haunts him.

All right, it happens. Some are writing the 5th build system already.

First, a small introduction, for those who joined recently.

[IX](https://github.com/stal-ix/ix) is a package manager for linux/macos and a Linux distribution based on it.

I really want as many people as possible to use the package manager (not right now, but in general), because 1 person will pull 500 packages (I have 300 now),  for more packages more people are needed.<br>
Therefore, I make a very clear distinction between a package manager that can be used anywhere and any way, even on another Linux, and a distribution built from those packages.<br>
Wherein, for example, I don’t want to lose potential users who, for some reason, like systemd (I generally think that adults, by mutual agreement, can also in #systemd, yes, although myself won’t for anything).

Therefore, I assume that I will have several "presets" - sets of packages that form a complete idea (type of linking + libc + DE + whatever).<br>
Further, when I say that "I have something like this in IX", I mean this preset here - [https://github.com/stal-ix/ix/blob/main/pkgs/set/system/0/ix.sh](https://github.com/stal-ix/ix/blob/main/pkgs/set/system/0/ix.sh), or System/0 #system0. Everything I do in it may not be relate to potential IX on #systemd + gnome.<br>
I'll repeat, I don’t care who and how uses the package database, as long as they help update the packages.

So, the intro is over. Further, about how System / 0 will be designed (or rather, about one of the implementation aspects).

I really wish that the package manager didn't run as root, and that it didn't have postinstall scripts. More precisely, I want the package to be able to be installed by unpacking the folder with its contents, and that's it. This greatly facilitates implementation, and simplifies the security model. Let's say like in Android.

I solved almost all problems related to this:

* adding users/groups
* resolver
* dynamic network setting

Yes, even per-package kernel settings.

But there was one problem that remained difficult for me. How to escalate privileges? Yes, sudo. If the package manager doesn’t work as root, there are no postinstall scripts, then how to get the #suid binary in the system?

At first, I solved this problem rather crookedly - a background process that set the setuid bit to some binaries.

Then I came up with an elegant solution, and I hasten to share it.

The host is running (say, via socket activation) an ssh daemon, as root. And sudo is a simple script that does ssh root@localhost. ssh is up on a unix domain socket so as not to shine on the network. Basically, this scheme is already working.

The more I think, the more I like this scheme:

* Privilege escalation in ssh is, I think, debugged even better than in sudo. Because so we escalate privileges IMHO more often;
* Login not by password, but by key, 1 per user, not one per host;
* Password/passwordless login remains on the user's side (password on the private part of the key), without fragile interference in /etc.

The scheme works very well - I don't have any #suid binaries.

- - -

## About bootstrapping
<sup> October 2, 2021 </sup>

#bootstrap

I am very interested in build systems, in the most diverse forms. So much so that I have:

1) My own build system for C++ code
2) And my own OSS packages build system. For Darwin and for Linux, for the latter it’s not only a packages build system, but a full-blown distribution:
[stal/IX](https://github.com/stal-ix)

I’ll talk about these projects here again, especially about the second one. For now I’ve referenced them as evidence of my great interest in assembly, distrobuilding and bootstrap topics.

Today I want to talk about the bootstrap problem.

What is it? Imagine that you’ve got on a desert island, you have an empty computer (no OS), and a CD with source programs. It is needed to get a working OS from this so that it would not be so boring (the stewardess didn’t get to a desert island with you).

This problem has the following aspects:

1) Historical. How generally was passing the way from a computer with punched cards to modern workstations and tablets?
2) Purely practical. How to deploy a distribution having the minimal possible binary seed. Here is a very interesting study on how to pave the path from a minimal hypervisor of 200 instructions to a modern Linux distribution - [https://github.com/fosslinux/live-bootstrap/blob/master/parts.rst](https://github.com/fosslinux/live-bootstrap/blob/master/parts.rst).
3) Security. A reproducible build with a minimum binary seed is needed to make sure that there are no Trojans in the software. For example, the notorious Thompson attack: [https://www.cs.cmu.edu/~rdriley/487/papers/Thompson_1984_ReflectionsonTrustingTrust.pdf](https://www.cs.cmu.edu/~rdriley/487/papers/Thompson_1984_ReflectionsonTrustingTrust.pdf).<br>
Very funny version - [https://www.quora.com/What-is-a-coders-worst-nightmare/answer/Mick-Stute?share=1](https://www.quora.com/What-is-a-coders-worst-nightmare/answer/Mick-Stute?share=1).<br>
And just to understand that your build of Alacritty is compiled from GitHub’s sources, and does not contain malware, or embedded microcode bug, is very valuable. I try not to keep software for which it is impossible to stretch such a chain from ground truth. 

There are several communities on the Internet who are also concerned about this problem:

1) [https://bootstrappable.org/](https://bootstrappable.org/)
2) [https://guix.gnu.org/](https://guix.gnu.org/) (very hard on bootstrap)
3) [https://nixos.org/](https://nixos.org/) (to a lesser extent, feel free to use Rust binary builds, for example)

Fellows from Guix even developed a couple of specialized languages for the initial bootstrap [https://www.gnu.org/software/mes/](https://www.gnu.org/software/mes/), [https://guix.gnu.org/blog/2019/guix-reduces-bootstrap-seed-by-50/](https://guix.gnu.org/blog/2019/guix-reduces-bootstrap-seed-by-50/). Guix, however, has other troubles of its own, so I can’t recommend it for everyday use. Nix is more interesting in this regard, I used it as a package manager for any Linux, Darwin, until I completed the bootstrap task of my own distribution.

What else is worth telling about the bootstrap:

* Ain’t all languages can be bootstrap. This means that some languages require a large initial blob of unknown quality. I try not to use such languages [https://elephly.net/posts/2017-01-09-bootstrapping-haskell-part-1.html](https://elephly.net/posts/2017-01-09-bootstrapping-haskell-part-1.html).

* Java becomes bootstrappable [https://bootstrappable.org/projects/jvm-languages.html](https://bootstrappable.org/projects/jvm-languages.html). The build system for it (Gradle) is still no.

* Rust, from my point of view, is bootstrappable with great reserve. Because: #mrustc:
1) To build the nth version, you need the n-1 version. This greatly increases the length of the bootstrap chain. About 20 builds at the moment?
2) The Standard ml first versions sources are unknown where. Bootstrap is only possible with a third party product [https://github.com/thepowersgang/mrustc](https://github.com/thepowersgang/mrustc).
3) The chain cannot be repeated on the Apple M1 yet.

* Go is well with bootstrap, the authors support version 1.4, the latest on C. You can build a modern Go with it. Also there is gccgo.

* Lisp is also very good. Let's say I ran the chain from the interpreted ECL to the compiled SBCL on the Apple M1 in an hour.

