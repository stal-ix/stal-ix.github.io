Hi!

Today I am decide to try IX as a standalone package manager.

I have:

Small libvirt KVM with freshly installed Debian (only ssh + standart tools was installed from debian-12.7.0-amd64-netinst.iso), 28 CPU, 64K RAM, 100Gb ssd, 1 NIC with NAT


After login to VM let's do some preparations:
```
$ mkdir ~/ix_root
$ export IX_ROOT=${HOME}/ix_root
$ sudo apt install g++
$ sudo apt install git
```
One can use the stable version of IX
```
$ git clone https://github.com/stal-ix/ix
```
But I'll use dev
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
 GNU nano, версия 7.2
 (C) 2023 the Free Software Foundation and various contributors
 Параметры сборки: --enable-utf8
```
And debian's nano:
```
$ nano --version
 GNU nano, версия 7.2
 (C) 2023 the Free Software Foundation and various contributors
 Параметры сборки: --disable-libmagic --enable-utf8
```
As we can see we can do it on any system with regular user and with only g++ and git installed

Have a nice day :0)

P.S. First run of *$ ix run bin/nano -- nano --version* took some time to build all the tools IX need in