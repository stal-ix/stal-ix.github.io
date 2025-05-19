# Caveats

> This document contains a regularly updated list of warnings and cautions. Please pay attention and do not miss them.

<!-- {% raw %} -->

The **main ALSA channel** can be muted after booting, adjust the volume with alsamixer and don't forget to unmute the channel (using M).

**Volume settings** are not saved after reboot. Add to your session script (e.g. .profile):

```shell
amixer sset Master unmute
amixer sset Master 100%
```

**Logs** are not saved after reboot.

**iwctl passwords** are not saved after reboot.

In fact, **any global state** is not saved after reboot, the system is now completely stateless. A suitable solution must be found for this problem.

There is **no logrotate for logs** in /var/run/, but they are also not saved after reboot, so that's not a real issue at the moment.

The **/ix/build folder is cleaned every 1000 seconds** ([https://github.com/stal-ix/ix/blob/main/pkgs/bin/sched/builddir/ix.sh#L11](https://github.com/stal-ix/ix/blob/main/pkgs/bin/sched/builddir/ix.sh#L11)), so to troubleshoot build issues, do one of the following:
* copy the build directory to another location;
* or, disable this scheduler altogether.

The **ACPI** rules are very rough, in fact they are only activated when the lid is closed/opened.

**Power management** is very basic. For now, we're running the CPU at full speed, because Linux governors are a mess.

**/etc/services**, despite being part of a read-only realm, is currently writable by the init.

**/dev protection** is weak - [https://github.com/stal-ix/ix/blob/main/pkgs/bin/mdevd/runit/conf/bad.conf](https://github.com/stal-ix/ix/blob/main/pkgs/bin/mdevd/runit/conf/bad.conf).

The **IX runtime** is not secure enough - runtime nodes have full access to /, and network isolation is not currently properly enforced.

The **service supervisor** is more like a stub than a real program - [https://github.com/stal-ix/ix/blob/main/pkgs/bin/runsrv/scripts/srv](https://github.com/stal-ix/ix/blob/main/pkgs/bin/runsrv/scripts/srv).

<!-- {% endraw %} -->
