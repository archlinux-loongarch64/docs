# 在 loongarch64 上面移植 archlinux

## 想法

龙芯 3a5000 采用了我国自主研制的 loongarch 架构，本人很喜欢 archlinux，所以尝试移植 archlinux 到龙芯 loongarch 架构上。

## 环境

CPU: AMD Ryzen 9 3900X

## 自己想到的做法

由于自己现在暂时还没有购买 3a5000 的电脑，所以暂时使用 qemu-loongarch64 进行编译。

### 做法

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

### 接下来

4. 尝试构建 基础系统
    * 参考资料
        * [build in a clean chroot](https://wiki.archlinux.org/title/DeveloperWiki:Building_in_a_clean_chroot)



## 抱怨

qemu-loongarch64 实在是太慢了 :-( ，如果有谁愿意提供一台龙芯电脑我感激不尽。

## 参考资料

[孙海勇老师的 clfs](https://github.com/sunhaiyong1978/CLFS-for-LoongArch)

[archlinux wiki](https://wiki.archlinux.org/)

## 感谢

[archlinux](https://www.archlinux.org/)

[孙海勇老师的 clfs](https://github.com/sunhaiyong1978/CLFS-for-LoongArch)
