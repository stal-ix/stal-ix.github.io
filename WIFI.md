```shell
ix# ix mut bin/iwd/ctl # install iwctl into user realm
ix# iwctl
> device list
...wlan0...
> station wlan0 scan
> station wlam0 get-networks
...<SSID>...
> station wlan0 connect <SSID>
```
