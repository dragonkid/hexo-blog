---
title: Streaming Media Server on Raspberry Pi
---

Deploy streaming media server on Raspberry Pi with Arch linux.

# Streaming Media Server on Raspberry Pi

## Mount Disk

增加对 NTFS 格式的支持：

```
pacman -S ntfs-3g
```

创建挂载目录并挂载移动硬盘：

```
mkdir /mnt/external_disk
mount /dev/sda1 /mnt/external_disk/
```

将移动硬盘设置为开机自动挂载：

```
echo -ne '/dev/sda1\t/mnt/external_disk/\tntfs-3g\tdefaults,noexec,umask=0000\t0\t0' >> /etc/fstab
```

## Minidlna

安装配置 minidlna，提供流媒体服务。

```
pacman -S minidlna
```

编辑配置文件 `/etc/minidlna.conf`，并修改如下配置：

```
network_interface=wlan0
root_container=B
friendly_name=xxxx
media_dir=V,/mnt/external_disk/Videos
```

还需配置如下参数，否则可能导致 inotify 构建播放列表失败，重启后生效。

```
echo 'fs.inotify.max_user_watches = 100000' >> /etc/sysctl.d/90-inotify.conf
```

设置 minidlna 开机启动 & 启动服务：

```
systemctl enable minidlna
systemctl start minidlna
```

minidlna 默认没有提供对 rm/rmvb 类型视频文件的支持，需修改代码后手工编译。

使用 minidlna 1.2.1 版本，修改代码如下：

metadata.c

```
else if( strncmp(ctx->iformatctx->name, "matroska", 8) == 0 )  
    xasprintf(&m.mime, "video/x-matroska");  
else if( strcmp(ctx->iformatctx->name, "flv") == 0 )  
    xasprintf(&m.mime, "video/x-flv");  
//----add----  
else if( strcmp(ctx->iformat->name, "rm") == 0 )  
    xasprintf(&m.mime, "video/x-pn-realvideo");  
else if( strcmp(ctx->iformat->name, "rmvb") == 0 )  
    xasprintf(&m.mime, "video/x-pn-realvideo");  
//----end---- 
if( m.mime )  
    goto video_nodlna; 
```

upnpglobalvars.h

```
"http-get:*:audio/mp4:*," \  
"http-get:*:audio/x-wav:*," \  
"http-get:*:audio/x-flac:*," \  
"http-get:*:application/ogg:*," \  
//----add----  
"http-get:*:video/x-pn-realvideo:*"  
//----end----  
```

utils.c

```
ends_with(file, ".m2t") || ends_with(file, ".mkv")   ||  
ends_with(file, ".vob") || ends_with(file, ".ts")    ||  
ends_with(file, ".flv") || ends_with(file, ".xvid")  ||  
//----add----  
ends_with(file, ".rm")  || ends_with(file, ".rmvb")  ||  
//----end----  
#ifdef TIVO_SUPPORT  
ends_with(file, ".TiVo") ||  
#endif  
ends_with(file, ".mov") || ends_with(file, ".3gp"));
```

手工编译安装步骤如下：

```
# 包依赖
pacman -S ffmpeg libexif libjpeg libid3tag flac libvorbis sqlite
./autogen.sh
./configure --prefix=/usr --sbindir=/usr/bin
make && make install

# 解决 fatal error: wand/MagickWand.h: No such file or directory 问题
pacman -S pkg-config
ln -sf /usr/include/ImageMagick-7/MagickWand /usr/include/ImageMagick-7/wand
```

## Samba

安装配置 samba 服务，便于上传文件。

```
pacman -S samba
```

编辑配置文件：

```
vi /etc/samba/smb.conf

[global]
        workgroup = WORKGROUP
        netbios name = XiaoZhuTVSmb
        server string = XiaoZhuTV Samba server
        security = user
[Movies]
        path = /mnt/external_disk/Videos
        valid users = god
        public = no
        writeable = yes
        browseable = yes
```

设置 samba 服务开机启动 & 启动服务：

```
systemctl start smbd.service
systemctl enable smbd.service
```




