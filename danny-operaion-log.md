# 在 loongarch64 上面移植 archlinux

## 想法

龙芯 3a5000 采用了我国自主研制的 loongarch 架构，本人很喜欢 archlinux，所以尝试移植 archlinux 到龙芯 loongarch 架构上。

## 环境

CPU: AMD Ryzen 9 3900X

## 自己想到的做法

由于自己现在暂时还没有购买 3a5000 的电脑，所以暂时使用 qemu-loongarch64 进行编译。

2022-03-27日，购买的龙芯电脑到货了。

### 做法

#### 2022-03-14

1. 安装好 qemu-loongarch64 后，使用 clfs 作为基准系统， chroot 进入 qemu 环境。
    * 参考资料
        * [qemu-loongarch64](https://github.com/sunhaiyong1978/CLFS-for-LoongArch/blob/main/Qemu_For_LoongArch64.md)
2. 下载 pacman 代码，参考 archlinux core 中的 pacman PKGBUILD 中的编译步骤进行编译，发现缺少 libarchive，下载编译 libarchive 进行编译，
  运行 makepkg 发现没有 fakeroot， 下载编译 fakeroot
    * 参考资料
        * [pacman PKGBUILD](https://github.com/archlinux/svntogit-packages/blob/master/pacman/trunk/PKGBUILD)
    * 遇到的问题和解决方案
        * libarchive 默认编译到 `/usr/lib` 而 archlinux 中的 `/usr/lib` 和 `/usr/lib64` 是软连接，但是 clfs 中，两个文件夹是相对独立的，
        所以编译的时候应该设置 libdir 为 `/usr/lib64`
        * fakeroot 编译失败，根据报错是 fakeroot 中有部分代码重复定义了 glibc 的代码。注释代码成功编译。
        * 关键代码位置
            * `#if __GLIBC_PREREQ(2,33)`
            * `#define WRAP_MKNOD mknod`
            * `#define WRAP_MKNODAT mknodat`
    * 疑问
        * fakeroot 编译失败是 fakeroot 问题还是 glibc 问题？
3. 使用 `pacvis`，在 x86_64 中查看所有包的依赖关系，了解依赖关系。

#### 2022-03-27

我购买的龙芯电脑到货啦！！！！

#### 2022-03-28

1. 下载孙海勇老师的 clfs， 划分一个单独的磁盘并格式化为 ext4，解压 clfs 到该磁盘中，复制 `/boot/efi/BOOTLOONGARCH.efi` 到一个 uefi shell 支持的磁盘中。
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
5. 稍等片刻成功进入系统，进入之后 tty1 是不能使用的，可以 `ctrl+alt+F2` 进入 `tty2` 即可正常使用。
6. `sudo systemctl start sshd` 可开启远程 ssh, 添加用户设置密码设置 sudo 然后就可远程 ssh 连接进入该电脑。

    * 参考资料
        * [UEFI Shell常用命令](http://lixingcong.github.io/2018/06/12/uefi-shell/)
        * [怎样修复grub开机引导(grub rescue)](https://blog.csdn.net/seaship/article/details/96427401)

#### 2022-03-28 - 2022-04-02

 1. 失败的操作



### 接下来

4. 尝试构建 基础系统
    * 参考资料
        * [build in a clean chroot](https://wiki.archlinux.org/title/DeveloperWiki:Building_in_a_clean_chroot)



## 抱怨

qemu-loongarch64 实在是太慢了 :-( ，如果有谁愿意提供一台龙芯电脑我感激不尽。

斥巨资购买了龙芯电脑 :-D.

## 参考资料

[孙海勇老师的 clfs](https://github.com/sunhaiyong1978/CLFS-for-LoongArch)

[archlinux wiki](https://wiki.archlinux.org/)

## 感谢

[archlinux](https://www.archlinux.org/)

[孙海勇老师的 clfs](https://github.com/sunhaiyong1978/CLFS-for-LoongArch)
