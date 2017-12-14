---
title: Arch on Raspberry Pi
---

Install arch to Raspberry Pi and the basic configuration of arch

# 安装

按照 [Arch Wiki](https://archlinuxarm.org/platforms/armv6/raspberry-pi) 中的步骤操作即可。

## FAQ

### mkfs.vfat: command not found

```
yaourt -S dosfstools    # for mkfs.vfat and mkfs.msdos
yaourt -S ntfsprogs     # for mkfs.ntfs
```

### Partition #1 contains a vfat signature

> Each disk and partition has some sort of signature and metadata/magic strings on it. The metadata used by operating system to configure disks or attach drivers and mount disks on your system. You can view such partition-table signatures/metadata/magic strings using the wipefs command. 

遇到上述提示选择 yes 即可。

## 网络

配置树莓派的 `ethernet` 网卡为固定 IP，这样就可以在树莓派启动后直接通过 `ethernet` 网卡直连树莓派，对其进行配置。

重新挂载 root 目录：

```
mount /dev/sdx2 root/
```

进入 root 目录，编辑/创建如下文件，并将其修改为如下内容：

```
vi etc/systemd/network/eth0.network
```

```
[Match]
Name=eth0

[Network]
Address=192.168.1.100/24
```

弹出 root 目录并保证数据正确写入：

```
umount root && sync
```

这样将就可以通过 `192.168.1.100` 这个 IP 与树莓派直连，并通过 ssh 进行登录完成后续配置过程。

# 配置

## 权限

切换到 root 用户：

```
su - root   # sudo 命令需要后续通过包管理工具安装
```

修改默认 alarm/root 用户密码：

```
passwd alarm
passwd root
```

## WIFI

无线网卡型号：TP-LINK TL-WN725N USB无线网卡

```shell
wifi-menu
```

设置无线自动连接

```
netctl enable wlan0-xxx
```


## 配置 locale

```
vi /etc/locale.gen
```

取消掉 en_US.UTF-8 和 zh_CN.UTF-8 的注释

```
locale-gen

echo LANG=en_US.UTF-8 > /etc/locale.conf
```

## 源

添加 archlinuxcn 源，在 `/etc/pacman.conf` 文件末尾加入如下内容：

```shell
[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = http://repo.archlinuxcn.org/$arch
```

测速并替换默认镜像源：

```
pacman -S reflector # arch arm 源中无此包
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
reflector -l 5 --sort rate --save /etc/pacman.d/mirrorlist
```

或直接添加清华源和中科大源：

```
Server = http://mirrors.tuna.tsinghua.edu.cn/archlinuxarm/$arch/$repo
Server = http://mirrors.ustc.edu.cn/archlinuxarm/$arch/$repo
```

安装 keyring：

```
pacman -S archlinuxcn-keyring
```

## yaourt

树莓派中需要手工编译

```
pacman -S yaourt 
```

## build-essential

```
pacman -S base-devel cmake sudo htop bmon wget tmux
```

# Backup

```
dd if=/dev/disk3 bs=16m | gzip > raspberrypi.img.gz
```

# Refs

* https://www.cyberciti.biz/faq/howto-use-wipefs-to-wipe-a-signature-from-disk-on-linux/

