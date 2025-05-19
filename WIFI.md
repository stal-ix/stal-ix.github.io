# Wi-Fi

Quick way:

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

or, alternatively, [a detailed description](https://wiki.archlinux.org/title/Iwd#iwctl)
