after boot main alsa channel can be muted, adjust volume with alsamixer, and don't forget to unmute channel (with M)

volume settings don't survive reboot

logs doesn't survive reboot

iwctl passwords doesn't survive reboot

actually, any global state don't survive reboot, system completely stateless by now, need to investigate proper solution for this

no logrotate for logs in /var/run/, but, they also don't survive reboot, so, this is not a real problem for now


