# サーバー構築実習（CentOS7）

## ファイル共有

### FTPサーバ構築

#### FTPとは

FTP(File Transfer Protocol)は、`クライアント`-`サーバ`間で、ファイルの送受信を行うための、通信プロトコルの一つです。利用方の一つとして、ホームページのデータなどを制作用PCからWebサーバに転送（アップロード）することです。また、ファイルのダウンロードサイトにて、ファイルのダウンロードを提供するために利用されています。

### FTPサーバの導入

#### vsftpdのインストール

```shell
[root@localhost ~]# yum -y install vsftpd
```

```shell
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  インストール中          : vsftpd-3.0.2-29.el7_9.x86_64                           1/1 
  検証中                  : vsftpd-3.0.2-29.el7_9.x86_64                           1/1 

インストール:
  vsftpd.x86_64 0:3.0.2-29.el7_9                                                       

完了しました!
```

```shell
[root@localhost ~]# systemctl start vsftpd.service
```

```shell
[root@localhost ~]# systemctl enable vsftpd.service
```

```shell
[root@localhost ~]# systemctl status vsftpd.service
```


```shell
 vsftpd.service - Vsftpd ftp daemon
   Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; enabled; vendor preset: disabled)
   Active: active (running) since 月 2022-11-21 15:54:56 JST; 7min ago
  Process: 4299 ExecStart=/usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf (code=exited, status=0/SUCCESS)
 Main PID: 4300 (vsftpd)
   CGroup: /system.slice/vsftpd.service
           └─4300 /usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf

11月 21 15:54:56 localhost.localdomain systemd[1]: Starting Vsftpd ftp daemon...
11月 21 15:54:56 localhost.localdomain systemd[1]: Started Vsftpd ftp daemon.
```

```shell
[root@localhost ~]# vi /etc/vsftpd/vsftpd.conf
```


```shell
# You may activate the "-R" option to the builtin ls. This is disabled by
# default to avoid remote users being able to cause excessive I/O on large
# sites. However, some broken FTP clients such as "ncftp" and "mirror" assume
# the presence of the "-R" option, so there is a strong case for enabling it.
#ls_recurse_enable=YES
#
# When "listen" directive is enabled, vsftpd runs in standalone mode and
# listens on IPv4 sockets. This directive cannot be used in conjunction
# with the listen_ipv6 directive.
listen=YES
#
# This directive enables listening on IPv6 sockets. By default, listening
# on the IPv6 "any" address (::) will accept connections from both IPv6
# and IPv4 clients. It is not necessary to listen on *both* IPv4 and IPv6
# sockets. If you want that (perhaps because you want to listen on specific
# addresses) then you must run two copies of vsftpd with two configuration
# files.
# Make sure, that one of the listen options is commented !!
listen_ipv6=NO

pam_service_name=vsftpd
userlist_enable=YES
userlist_deny=NO
tcp_wrappers=YES

pasv_min_port=4000
pasv_max_port=4010
```


```shell
[root@localhost ~]# cat /etc/vsftpd/ftpusers
```

```shell
# Users that are not allowed to login via ftp
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
news
uucp
operator
games
nobody
```

```shell
[root@localhost ~]# vi /etc/vsftpd/user_list
```


```shell
# vsftpd userlist
# If userlist_deny=NO, only allow users in this file
# If userlist_deny=YES (default), never allow users in this file, and
# do not even prompt for a password.
# Note that the default vsftpd pam config also checks /etc/vsftpd/ftpusers
# for users that are denied.
#root
#bin
#daemon
#adm
#lp
#sync
#shutdown
#halt
#mail
#news
#uucp
#operator
#games
#nobody
user
```

```shell
[root@localhost ~]# systemctl restart vsftpd.service
```


```shell
[root@localhost ~]# firewall-cmd --add-service=ftp --zone=public --permanent
```

```shell
[root@localhost ~]# firewall-cmd --add-port=4000-4010/tcp --zone=public --permanent
```

```shell
[root@localhost ~]# firewall-cmd --reload
```

```shell
[root@localhost ~]# firewall-cmd --list-all
```


```shell
[root@localhost ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens33
  sources: 
  services: ssh dhcpv6-client ftp
  ports: 4000-4010/tcp
```

#### FFFTPのインストール

### ファイルサーバ構築

#### ファイルサーバとは

オフィスや家庭で複数のPCから共同で作業を行いたいとき、LAN経由としてファイルを共有できると便利です。このときに用いるサーバを`「ファイルサーバ」`といいます、LAN上にある共有ハードディスクと捉えるとよいでしょう。多くのクライアントPCはWindowsで動作しています。Windowsは、標準でファイルサーバとなる機能やそれを利用する機能があります。Linuxでは、Sambaと呼ばれるソフトウェアが提供されており、これを利用すると、Linuxサーバ上にファイルサーバを構築して、Windowsから利用することができます。

#### Sambaのインストール

