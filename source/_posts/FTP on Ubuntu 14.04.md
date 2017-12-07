---
title: FTP on Ubuntu 14.04
---

Install and config vsftp on ubuntu 14.04

# FTP on Ubuntu 14.04

## install

```
apt-get install vsftpd
```

备份默认配置文件

```
cp /etc/vsftpd.conf /etc/vsftpd.conf.orig
```

## iptables

We'll need to open ports 20 and 21 for FTP, port 990 for later when we enable TLS, and ports 40000-50000 for the range of passive ports.

```
-A INPUT -i eth0 -p tcp -m tcp --dport 20 -j ACCEPT
-A INPUT -i eth0 -p tcp -m tcp --dport 21 -j ACCEPT
-A INPUT -i eth0 -p tcp -m tcp --dport 990 -j ACCEPT
-A INPUT -i eth0 -p tcp -m tcp --dport 40000:50000 -j ACCEPT
```

## User Directory

添加用户，用户名自定义，设置密码后一路回车。

```
useradd transfer
```

FTP is generally more secure when users are restricted to a specific directory.vsftpd accomplishes this with chroot jails. When chroot is enabled for local users, they are restricted to their home directory by default. However, because of the way vsftpd secures the directory, it must not be writable by the user. This is fine for a new user who should only connect via FTP, but an existing user may need to write to their home folder if they also shell access.

In this example, rather than removing write privileges from the home directory, we're will create an ftp directory to serve as the chroot and a writable files directory to hold the actual files.

```
mkdir -p /home/transfer/ftp
chown nobody:nogroup /home/transfer/ftp
chmod a-w /home/transfer/ftp
```

Next, we'll create the directory where files can be uploaded and assign ownership to the user.

```
mkdir -p /home/transfer/ftp/files
chown transfer:transfer /home/transfer/ftp/files/
```

## Configuring FTP Access

编辑文件 `/etc/vsftpd.conf`，修改/添加如下配置后保存并退出。

```
write_enable=YES
chroot_local_user=YES

pasv_min_port=40000
pasv_max_port=50000

# We’ll add a user_sub_token in order to insert the username in our local_root directory path so our configuration will work for this user and any future users that might be added.
user_sub_token=$USER
local_root=/home/$USER/ftp

# Since we’re only planning to allow FTP access on a case-by-case basis, we’ll set up the configuration so that access is given to a user only when they are explicitly added to a list rather than by default.
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO
```

添加 ftp 用户到 vsftpd.userlist。

```
echo "transfer" | tee /etc/vsftpd.userlist
```

## Restart vsftpd

```
service vsftpd restart
```

## Refs

* https://www.digitalocean.com/community/tutorials/how-to-set-up-vsftpd-for-a-user-s-directory-on-ubuntu-16-04



