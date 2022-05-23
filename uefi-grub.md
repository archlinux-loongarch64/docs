
## 用 efi 文件引导 grub

1. 复制 `/boot/efi/BOOTLOONGARCH.efi` 到一个 uefi shell 支持的磁盘中。
2. 进入 uefi shell 添加 efi 文件为启动项，重启选择该启动项。

```UEFI
> fs0:
> bcfg boot add 8 fs0:\EFI\bootloongarch.efi "loongarch-clfs"
```

3. 进入 grub rescue 查找 grub

```grub rescue
grub rescue> root=(hd1,gpt4)
grub rescue> prefix=/boot/grub
grub rescue> set root=(hd1,gpt4)
grub rescue> set prefix=(hd1,gpt4)/boot/grub
grub rescue> insmod normal
grub rescue> normal
```

4. 进入 GRUB 进行手动引导

```grub
grub> set root=(hd1,gpt4)
grub> set prefix=(hd1,gpt4)/boot/grub
grub> linux /boot/vmlinux root=/dev/nvme0n1p4
grub> initrd /boot/acpi-initrd
grub> boot
```