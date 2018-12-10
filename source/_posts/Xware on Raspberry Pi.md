---
title: Xware on Raspberry Pi
---

Use Raspberry Pi as Thunder remote downloader.

# Xware on Raspberry Pi

Xware 是迅雷开发的一款远程下载固件，可以使用其在树莓派上自行搭建迅雷远程下载器。

## Install Xware

```
take /usr/share/thunder
wget -c https://raw.githubusercontent.com/snail007/xware/master/Xware_armel_v5te_glibc.tar.gz
tar zxvf Xware_armel_v5te_glibc.tar.gz
```

## Active Code

```
[root@alarmpi thunder]# ./portal
initing...

try stopping xunlei service...
killall: ETMDaemon: no process killed
killall: EmbedThunderManager: no process killed

setting xunlei runtime env...
port: 9000 is usable.

YOUR CONTROL PORT IS: 9000


starting xunlei service...
execv: /root/thunder/lib/ETMDaemon.

getting xunlei service info...
Connecting to 127.0.0.1:9000 (127.0.0.1:9000)
xunlei_portal.tmp        0T --:--:-- ETA
the active key is not valid.

try again...(has tried 1 time(s)).
getting xunlei service info...
Connecting to 127.0.0.1:9000 (127.0.0.1:9000)
xunlei_portal.tmp        0T --:--:-- ETA

THE ACTIVE CODE IS: abcdef

go to http://yuancheng.xunlei.com, bind your device with the active code.
finished.
```

其中 `THE ACTIVE CODE IS` 后的既是设备激活码。

## Activate Downloader

登陆 `http://yuancheng.xunlei.com/`，添加下载器，输入激活码绑定即可。

## Config Download Path

Xware 默认使用已经存在的挂载点作为下载目录，该下载目录也可以自行配置。

创建新的挂载点：

```
mount --bind /data/TDDOWNLOAD /mnt/TDDOWNLOAD
```

在 `~/thunder` 目录下创建 `etc` 目录，编辑文件 `thunder_mounts.cfg` 如下：

```
avaliable_mount_path_pattern
{
    /mnt/TDDOWNLOAD
}
```

重启路由器固件即可。

## Auto Start

```
vi /usr/lib/systemd/system/thunder-portal.service

[Unit]
Description=thunder remote downloader
After=mnt-external_disk.mount network.service

[Service]
Type=oneshot
User=root
Group=root
ExecStart=/usr/share/thunder/portal
RemainAfterExit=true
ExecStop=/usr/share/thunder/portal
ProtectHome=on
NoNewPrivileges=on

[Install]
WantedBy=multi-user.target
```

```
systemctl daemon-reload
systemctl start thunder-portal.service && systemctl enable thunder-portal.service
```




