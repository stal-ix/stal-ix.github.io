# Caveats

> This document contains a regularly replenishing list of warnings and cautions. Pay attention, don't miss it out.

**Main alsa channel** after boot can be muted, adjust volume with alsamixer, and don't forget to unmute channel (with M).

**Volume settings** don't survive reboot. Add to your session script (e.g. .profile):

```shell
amixer sset Master unmute
amixer sset Master 100%
```

**Logs** doesn't survive reboot.

**iwctl passwords** doesn't survive reboot.

**Any global state**, actually, don't survive reboot, system completely stateless by now, need to investigate proper solution for this.

**No logrotate for logs** in /var/run/, but, they also don't survive reboot, so, this is not a real problem for now.

**/ix/build folder trashed every 1000 seconds** [https://github.com/stal-ix/ix/blob/main/pkgs/bin/sched/builddir/ix.sh#L11](https://github.com/stal-ix/ix/blob/main/pkgs/bin/sched/builddir/ix.sh#L11), so, to debug build problems, do one of the following:  
* copy build dir somewhere else;
* or, disable this scheduler altogeter.

**ACPI** rules very approximate, actually, activated only on lid close/open.

**Power management** is very basic . For now, we're running CPU at full speed, because linux governors are a mess.

**/etc/services**, despite being part of read-only realm, are currently writeable for runit.

**/dev/* protection** is weak - [https://github.com/stal-ix/ix/blob/main/pkgs/bin/mdevd/runit/scripts/bad.conf](https://github.com/stal-ix/ix/blob/main/pkgs/bin/mdevd/runit/scripts/bad.conf).

**IX runtime** are not safe enough - execution nodes have full access to /, and network isolation currently not really enforced.

**Service supervisor** is more like a stub than a real program - [https://github.com/stal-ix/ix/blob/main/pkgs/bin/runsrv/srv](https://github.com/stal-ix/ix/blob/main/pkgs/bin/runsrv/srv).
