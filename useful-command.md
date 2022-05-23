# 常用命令

### 挂载

```
sudo mount /dev/nvme0n1p3 /mnt/nvme0n1p3/
sudo mount /dev/nvme0n1p4 /mnt/lfs/
sudo mount /dev/nvme0n1p5 /mnt/nvme0n1p5

sudo mount -v --bind /dev $LFS/dev
sudo mount -v --bind /dev/pts $LFS/dev/pts
sudo mount -vt proc proc $LFS/proc
sudo mount -vt sysfs sysfs $LFS/sys
sudo mount -vt tmpfs tmpfs $LFS/run
```

### 卸载

```
sudo umount $LFS/run
sudo umount $LFS/sys
sudo umount $LFS/proc
sudo umount $LFS/dev/pts
sudo umount $LFS/dev
```

### chroot

```
sudo chroot "$LFS" /usr/bin/env -i   \
    HOME=/root                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u: \w \$ ' \
    PATH=/usr/bin:/usr/sbin \
    /bin/bash --login


sudo chroot  --userspec=1001:1002 "$LFS" /usr/bin/env -i   \
    HOME=/home/lfsuser                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u: \w \$ ' \
    PATH=/usr/bin:/usr/sbin \
    /bin/bash --login

```

### alias

```
export MAKEFLAGS='-j5'
alias mm='makepkg --skippgpcheck  --nocheck -A -C -d  -f'
alias mmo='makepkg --skippgpcheck  --nocheck -A -C -d  -f -o'
alias mme='makepkg --skippgpcheck  --nocheck -A -C -d  -f -e'
alias pp='pacman -U *.pkg.tar'
alias ppd='pacman -Udd *.pkg.tar'
alias ppa='pacman -Udd *.pkg.tar --overwrite "*" '


makepkg --skipchecksums --skipinteg --skippgpcheck  -A --nocheck -C -d  -f -e


```


### 常见操作

```
asp checkout filesystem
cd filesystem/repos/core-x86_64
# 做相关更改
makepkg -A -d
pacman -U -r /mnt/lfs filesystem-2021.12.07-1-loongarch64.pkg.tar
```

```
find -name 'config.sub' -exec echo {} \;  -exec cp /sources/config.sub {} \;
find -name 'config.guess' -exec echo {} \;  -exec cp /sources/config.guess {} \;
```

### 其他常见命令

```
find -name "*.pkg.tar*" -exec cp -nv {} /mnt/nvme0n1p5/apkg/ \;
```