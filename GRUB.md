# GRUB

This text assumes that you know what GRUB is, what it is used for, and have a general understanding of how a computer boots up.

(1) [https://wiki.archlinux.org/title/GRUB](https://wiki.archlinux.org/title/GRUB)<br>
(2) [https://wiki.archlinux.org/title/EFI_system_partition](https://wiki.archlinux.org/title/EFI_system_partition)<br>
(3) [https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface)<br>

## UEFI

Become root user:
```
$ sudo sh
```

Mount the pseudo filesystem with EFI variables:


```
# mount -t efivarfs efivarfs /sys/firmware/efi/efivars
```

If an error occurs at this stage, you need to boot from external media in UEFI mode.

Mount the EFI partition (if it does not exist, the process of creating it is well described in (2)):
```
# mkdir -p /esp
# mount {{your_efi_partition}} /esp
```

Install the necessary tools:
```
# ix mut set/boot/efi
```

Install the bootloader:
```
# grub-install --target=x86_64-efi --efi-directory=/esp --bootloader-id=GRUB
```

Configure GRUB to automatically detect all kernels installed in the system realm:
```
# cat << EOF > /boot/grub/grub.cfg
configfile /etc/grub.cfg
EOF
# ix mut system bin/kernel/gengrub
```

## Legacy BIOS

Become root user:
```
$ sudo sh
```

Install the GRUB bootloader package:
```
# ix mut bin/grub/bios
```

Install the bootloader itself:
```
# grub-install --target=i386-pc /dev/xxx
```

Configure GRUB to automatically detect all kernels installed in the system realm:
```
# cat << EOF > /boot/grub/grub.cfg
configfile /etc/grub.cfg
EOF
# ix mut system bin/kernel/gengrub
```
