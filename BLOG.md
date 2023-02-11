# stal/IX blog

> About building systems in a different way, static linking, and much more!<br>
> And about the stal/IX, of course, from the benevolent dictator (@pg83).

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

