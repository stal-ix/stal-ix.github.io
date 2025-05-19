# Using IX as a standalone package manager

> Prerequisites:<br>
> [IX.md](IX.md)<br>

You can use IX as a standalone package manager.

Let's make some preparations:
```
$ mkdir ~/ix_root
$ export IX_ROOT=${HOME}/ix_root
$ sudo apt install g++
$ sudo apt install git
```

You can use the stable branch of IX:
```
$ git clone https://github.com/stal-ix/ix
```
Or you can use the development branch:
```
$ git clone https://github.com/pg83/ix
```

Now we need our system to know about the IX binary path:
```
$ export PATH=${PWD}/ix:${PATH}
```

Let's try to check the version of nano from IX:
```
$ ix run bin/nano -- nano --version
TOUCH /home/spiage/ix_root/store/IVNnNzFRjwyXtgNF-rlm-ephemeral/touch
 GNU nano, version 7.2
 (C) 2023 the Free Software Foundation and various contributors
 Compiled options: --enable-utf8
```
And nano from Debian:
```
$ nano --version
 GNU nano, version 7.2
 (C) 2023 the Free Software Foundation and various contributors
 Compiled options: --disable-libmagic --enable-utf8
```
As you can see, this can be done on any system with a regular user, where only g++ and git are installed.

The first time you run `$ ix run bin/nano -- nano --version` takes some time because it needs to build all the necessary tools.
