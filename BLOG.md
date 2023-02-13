# stal/IX blog

> About building systems in a different way, static linking, and much more!<br>
> And about the stal/IX, of course, from the benevolent dictator (@pg83).

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

Separately, I note that these crutch package managers are completely shitty. I would really like to see how cargo, for example, tries to vendor any lib with settings and data for this lib.

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

