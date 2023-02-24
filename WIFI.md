quick and dirty

```shell
user# ix mut bin/iwd/ctl # install iwctl into user realm
user# iwctl
> device list
...wlan0...
> station wlan0 scan
> station wlam0 get-networks
...<SSID>...
> station wlan0 connect <SSID>
```

[detailed description](https://wiki.archlinux.org/title/Iwd#iwctl)
