# Using IX as a standalone package manager

> Prereq:<br>
> [IX.md](IX.md)<br>

You can use IX as a standalone package manager.

Let's do some preparations:
```
$ mkdir ~/ix_root
$ export IX_ROOT=${HOME}/ix_root
$ sudo apt install g++
$ sudo apt install git
```

One can use the stable branch of IX:
```
$ git clone https://github.com/stal-ix/ix
```
Or you can choose to use the development branch:
```
$ git clone https://github.com/pg83/ix
```

Now we need our system to know about IX binary path:
```
$ export PATH=${PWD}/ix:${PATH}
```

So let's try to check IX's nano version:
```
$ ix run bin/nano -- nano --version
TOUCH /home/spiage/ix_root/store/IVNnNzFRjwyXtgNF-rlm-ephemeral/touch
 GNU nano, version 7.2
 (C) 2023 the Free Software Foundation and various contributors
 Compiled options: --enable-utf8
```
And Debian's nano:
```
$ nano --version
 GNU nano, version 7.2
 (C) 2023 the Free Software Foundation and various contributors
 Compiled options: --disable-libmagic --enable-utf8
```
As you can see you can do it on any system with regular user and with only g++ and git installed.

First run of *$ ix run bin/nano -- nano --version* takes some time to build all the tools IX need in.
