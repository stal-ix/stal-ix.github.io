after boot main alsa channel can be muted, adjust volume with alsamixer, and don't forget to unmute channel (with M)

volume settings don't survive reboot. Add to your session script (e.g. .profile):

```shell
amixer sset Master unmute
amixer sset Master 100%
```

logs doesn't survive reboot

iwctl passwords doesn't survive reboot

actually, any global state don't survive reboot, system completely stateless by now, need to investigate proper solution for this

no logrotate for logs in /var/run/, but, they also don't survive reboot, so, this is not a real problem for now

```shell
user# cat /etc/sche.d/1000/builddir.sh 
/bin/flock -nx /ix /bin/sh -c 'mv /ix/build/* /ix/trash/'
```

/ix/build folder trashed every 1000 seconds or so, so, if one wants to debug build problems, it should do one of the:

* copy build dir somewhere else
* or, disable this scheduler altogeter

acpi rules very approximate, actually, activated only on lid close/open

very basic power management. For now, we run CPU full speed, because linux governors are mess

/etc/services, despite being part of read-only realm, are writeable for runit for now

/dev/* protection is weak - https://github.com/stal-ix/ix/blob/main/pkgs/bin/mdevd/runit/scripts/bad.conf

IX runtime are not safe enough - execution nodes have full access to /, and network isolation not really enforced for now

Service supervisor looks like more of a stub, than of real program - https://github.com/stal-ix/ix/blob/main/pkgs/bin/runsrv/srv


