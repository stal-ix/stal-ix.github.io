Этот текст предполагает, что вы знаете, что такое GRUB, для чего он нужен, и примерно представляете, как загружается компьютер.

(1) https://wiki.archlinux.org/title/GRUB
(2) https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface

Станем root:
```
$ sudo sh
```

Примонтируем псевдо fs c efi переменными: 
```
# mount -t efivarfs efivarfs /sys/firmware/efi/efivars
```

Если на этой стадии случилась ошибка - вам нужно загрузиться с внешнего media в uefi режиме.

Примонтируем EFI partition (если ее нет, то процесс ее создания хорошо описан в (2)):
```
# mkdir -p /esp
# mount {{your_efi_partition}} /esp
```

Установим нужные инструменты:
```
# ix mut set/boot/efi
```

Установим загрузчик:
```
# grub-install --target=x86_64-efi --efi-directory=/esp --bootloader-id=GRUB
```

Настроим GRUB так, чтобы он автоматически находил все ядра, которые вы установили в system realm:
```
# cat << EOF > /boot/grub/grub.conf
configfile /etc/grub.cfg
EOF
# ix mut system bin/kernel/gengrub
```
