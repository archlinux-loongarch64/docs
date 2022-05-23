# 在 loongarch64 上面移植 archlinux

## 想法

龙芯 3a5000 采用了我国自主研制的 loongarch 架构，本人很喜欢 archlinux，所以尝试移植 archlinux 到龙芯 loongarch 架构上。

## 环境

CPU: Loongarch64 3A5000LL 2.3Ghz

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

#### 2022-03-28 - 2022-04-02 失败的操作

##### 第一次失败

1. clfs 进入后，首先编译出 pacman asp，然后开始分析 archlinux 的依赖关系，将依赖关系和 clfs 中提到的顺序进行对照
2. 开始使用 makepkg 进行编译，编译顺序是 asp checkout filesystem tz-data iana-etc linux-api-headers glibc binutils gcc ncurse readline .....
    * 虽然说成功编译出来了 pkg.tar.gz 包，但是这些包都是不可用的，很多包中的 lib 包是依赖当前系统的。所以这种方案从最开始就是错的。

##### 第二次失败

1. clfs 进入后，编译出 pacman asp，分析 archlinux 依赖关系，将依赖关系和 clfs 进行对照
2. 与第一次失败不同的是，每编译一个包直接安装到当前系统。按照依赖关系从低到高依次编译
3. 编译顺序参考 linux-api-headers 中 pkgbuid 的顺序。 `linux-api-headers->glibc->binutils->gcc->glibc->binutils->gcc`
    * archlinux 的依赖关系中有很多的循环依赖，并且由于 kiss 的思想，archlinux 的 lib 库有一部分是和 bin 是在一个 pkgbuid 中的，造成编译困难
    * 编译到 systemd 的时候出现了严重的错误，这个错误我修复不了，而且当前系统已经被我替换了很多包，重启之后系统就挂掉了。

##### 反思

1. 其实最开始 shipujin 已经提醒我不要这么干，我也明白会出现这种问题，但是我觉得我还是要试一试。
2. LFS 的思想是首先编译一个 `交叉工具链和临时工具` 隔离宿主机，然后在使用这套工具进行编译，这样的话就能完全隔离宿主机。

#### 2022-04-02 - 2022-04-12

1. 参考 linux-api-headers 的 PKGBUILD 中编译顺序我们发现，这个顺序和 lfs 中顺序相同。
1. 查找资料，在 github 上找到了一篇关于添加 pacman 到 lfs 的介绍，开始构建自己的系统。
2. 按照 lfs 构建一个基本的系统，走到 `7.14 清理和备份临时系统` 之后开始构建 pacman.
3. 由于 pacman 最新版本采用的是 meson 进行构建，meson 依赖较多，所以采用文档中提到的老版本 pacman，构建完成 meson 之后再回来重新构建新版本 pacman。

    * 注意事项，遇到的问题
        * 遇到循环依赖时，按照 lfs 的顺序，适当修改 PKGBUILD 进行安装。
        * 对于第一遍第二遍使用的所有包版本必须要完全使用一样的。

    * 参考资料
        * [benvd/lfs-pacman](https://github.com/benvd/lfs-pacman)
        * [LFS 8.1](https://lfs-hk.koddos.net/museum/lfs-museum/8.1-systemd/LFS-BOOK-8.1-systemd.pdf)
        * [LFS 11](https://www.linuxfromscratch.org/lfs/read.html)
        * [用GCC-12最新版编译Linux Kernel需要用到的补丁](https://bbs.loongarch.org/d/49-gcc-12linux-kernel)

#### 2022-04-12

1. 在下午大约四点半左右，我即将编译完 linux 内核的时候，电脑死机，重启之后发现电脑内部一直响。经排查发现 ssd 损坏。我两个多星期的心血毁于一旦。
2. 联系了 ssd 厂家同意换新，现在正在返厂路上。
3. 所有的一切都要重来了。 :-(

#### 2022-04-21

新 ssd 到货

#### 2022-04-21 - 2022-05-23

1. 由于工作太忙中间有几天没有进行研究。
2. 将 ssd 装到另外一台电脑上，分区，下载 clfs，解压，asp checkout, git clone linux binutils glibc gcc 等。
3. 将 ssd 装到龙芯电脑上，uefi + grub 进行引导。
4. 按照原来的流程重新编译一遍。
5. 自己按照 archlinux pkgbuild 编译 linux 死机，查询资料可能由于内核固件不兼容问题导致。
6. 将 clfs 中的 acpi-initrd 和 vmlinux 复制到 /boot 中，将 linux 内核复制过来。
7. 复制 clfs 中的 grub 中启动配置，添加一个条目，进行引导启动，键盘可以使用，但是屏幕无法显示 卡在了 loading kernel。
8. 购买串口线进行抓 log.
9. 串口中的信息已经正确进入 login 页面，初步判断可能是显卡驱动相关的问题。

### 接下来

1. 对比 clfs 中串口和当前系统串口信息，看看是不是有什么固件没有启动。

    * 参考资料
        * [串口](https://bbs.loongarch.org/d/40-3a5000clfs)
        * [loongarch-next](https://bbs.loongarch.org/d/45-loongarch-next-linux)
        * [gcc-12 linux kernel](https://bbs.loongarch.org/d/49-gcc-12linux-kernel)

## 抱怨

qemu-loongarch64 实在是太慢了 :-( ，如果有谁愿意提供一台龙芯电脑我感激不尽。

斥巨资购买了龙芯电脑 :-D.

## 感谢

[archlinux](https://www.archlinux.org/)

[孙海勇老师的 clfs](https://github.com/sunhaiyong1978/CLFS-for-LoongArch)

[LA UOSC](https://bbs.loongarch.org/)
