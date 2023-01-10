# サーバー構築実習（CentOS7）

## FTPサーバ構築

### FTPとは

FTP(File Transfer Protocol)は、`クライアント`-`サーバ`間で、ファイルの送受信を行うための、通信プロトコルの一つです。利用方の一つとして、ホームページのデータなどを制作用PCからWebサーバに転送（アップロード）することです。また、ファイルのダウンロードサイトにて、ファイルのダウンロードを提供するために利用されています。

## １．FTPサーバ（vsftpd）の導入

### vsftpdのインストール

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

- vsftpdの起動

```shell
[root@localhost ~]# systemctl start vsftpd.service
```

- vsftpdの自動起動を有効化

```shell
[root@localhost ~]# systemctl enable vsftpd.service
```

- vsftpdのステータスを確認

```shell
[root@localhost ~]# systemctl status vsftpd.service
```

Active欄が`active(running)`になっていれば成功

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

### vsftpd.confの設定

```shell
[root@localhost ~]# vi /etc/vsftpd/vsftpd.conf
```

- 以下の内容を変更
  - listen=YES
  - listen_ipv6=NO
  - userlist_deny=NO
  - pasv_min_port=4000
  - pasv_max_port=4010

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

### ftpusersの設定

ftpusersの確認

```shell
[root@localhost ~]# cat /etc/vsftpd/ftpusers
```

ユーザ`user`が含まれていないことを確認する。このファイルに記載されてるユーザはFTP接続ができない。

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

### user_listの設定

```shell
[root@localhost ~]# vi /etc/vsftpd/user_list
```

`user`以外は全てコメント化させる。このファイルに記載されていユーザのみFTP接続が利用できる。

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

### FTPサービスの再起動

```shell
[root@localhost ~]# systemctl restart vsftpd.service
```

### ファイアウォールの設定

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

### FFFTPのインストール

### 動作確認

## ファイルサーバ構築

### ファイルサーバとは

オフィスや家庭で複数のPCから共同で作業を行いたいとき、LAN経由としてファイルを共有できると便利です。このときに用いるサーバを`「ファイルサーバ」`といいます、LAN上にある共有ハードディスクと捉えるとよいでしょう。多くのクライアントPCはWindowsで動作しています。Windowsは、標準でファイルサーバとなる機能やそれを利用する機能があります。Linuxでは、Sambaと呼ばれるソフトウェアが提供されており、これを利用すると、Linuxサーバ上にファイルサーバを構築して、Windowsから利用することができます。

## １．ファイルサーバ(Samba)の導入

### sambaのインストール

```shell
[root@file ~]# yum -y install samba
```

以下のように表示されれば成功

```shell
インストール:
  samba.x86_64 0:4.10.16-20.el7_9                                                                                       
依存性関連をインストールしました:
  pyldb.x86_64 0:1.5.4-2.el7                         pytalloc.x86_64 0:2.1.16-1.el7                      python-tdb.x86_64 0:1.3.18-1.el7           
  samba-common-libs.x86_64 0:4.10.16-20.el7_9        samba-common-tools.x86_64 0:4.10.16-20.el7_9        samba-libs.x86_64 0:4.10.16-20.el7_9       

依存性を更新しました:
  dbus.x86_64 1:1.10.24-15.el7                         dbus-libs.x86_64 1:1.10.24-15.el7               dbus-x11.x86_64 1:1.10.24-15.el7              
  libldb.x86_64 0:1.5.4-2.el7                          libsmbclient.x86_64 0:4.10.16-20.el7_9          libtalloc.x86_64 0:2.1.16-1.el7               
  libtdb.x86_64 0:1.3.18-1.el7                         libtevent.x86_64 0:0.9.39-1.el7                 libwbclient.x86_64 0:4.10.16-20.el7_9         
  samba-client-libs.x86_64 0:4.10.16-20.el7_9          samba-common.noarch 0:4.10.16-20.el7_9         

完了しました!
```

### smb.confの設定

```shell
[root@file ~]# vi /etc/samba/smb.conf
```

- 以下の内容を変更
  - workgroup = WORKGROUP

```conf
# See smb.conf.example for a more detailed config file or
# read the smb.conf manpage.
# Run 'testparm' to verify the config is correct after
# you modified it.

[global]
        workgroup = WORKGROUP
        security = user

        passdb backend = tdbsam

        printing = cups
        printcap name = cups
        load printers = yes
        cups options = raw
```

### ユーザの登録

すでに存在するLinuxユーザのうち、sambaによるファイル共有を利用するユーザをsambaユーザとして登録する。今回は、`user`を設定する。

```shell
[root@localhost ~]# smbpasswd -a user
```

LinuxユーザとSambaユーザでは、内部でパスワードを管理する方法が異なるため、Samba用にパスワードを設定する必要がある。今回は、混乱しないようにパスワードは、Linuxユーザのログインパスフレーズを利用する。

```shell
New SMB password:
Retype new SMB password:
Added user user.
```

### Sambaサービスの起動

```shell
[root@localhost ~]# systemctl start smb.service
```

```shell
[root@localhost ~]# systemctl start nmb.service
```

### Sambaサービスの自動起動の有効化

```shell
[root@localhost ~]# systemctl enable smb.service
```

```shell
[root@localhost ~]# systemctl enable nmb.service
```

### ファイアウォールの設定

```shell
[root@localhost ~]# firewall-cmd --add-service=samba --zone=public
[root@localhost ~]# firewall-cmd --add-service=samba --zone=public --permanent
```

### SELinuxの無効化

```shell
[root@localhost ~]# setenforce 0
```

### 動作確認

Windowsからファイルサーバにアクセスする。エクスプローラを起動し、アドレスバーに`￥￥サーバのIPアドレス`のように入力する。続いて、Samabaユーザのユーザ名とパスワードを入力する。

  - `user`ディレクトリをクリックすると、`user`のホームディレクトリにアクセス可能

  - Windowsから新規フォルダを作成し、Linux側の`user`のホームディレクトリを確認

    ```shell
    [user@ns1 ~]$ cd /home/user/
    [user@ns1 ~]$ ls -l
    ```
  
  - Linux側から新規ファイルを作成し、Windowsから確認

    ```shell
    [user@ns1 ~]$ cd /home/user/
    [user@ns1 ~]$ vim sample.txt
    ```

Windowsのコマンドプロンプトを開き、ファイル共有として接続しているユーザを確認する。

```shell
C:\Users\user>net use
```


## ２．複数ユーザ間での共有ディレクトリ

Sambaで複数のユーザが共有できるフォルダを設置する。

### 共有するフォルダの作成

オプション-pは、パスの途中に存在しないフォルダがあっても、自動的にそのフォルダを作成する。

```shell
[root@localhost ~]# mkdir -p /var/samba/share
```

### 共有フォルダの権限付与

```shell
[root@localhost ~]# chmod 777 /var/samba/share
```

#### ユーザの追加

```shell
[root@localhost ~]# useradd hirota
```

```shell
[root@localhost ~]# passwd hirota
ユーザー hirota のパスワードを変更。
新しいパスワード:
新しいパスワードを再入力してください:
passwd: すべての認証トークンが正しく更新できました。
[root@localhost ~]#
```

### smb.confの設定

```shell
[root@file ~]# vi /etc/samba/smb.conf
```

- 以下の内容を末尾に追加

  ```shell
  [share]
          comment = Share Directory
          path = /var/samba/share
          browseable = yes
          writable = yes
          valid users = user, hirota
          create mask = 0644
          directory mask = 0775
  ```

### Sambaサービスの起動

```shell
[root@localhost ~]# systemctl restart smb.service
```

```shell
[root@localhost ~]# systemctl restart nmb.service
```

### 動作確認

Windowsからファイルサーバにアクセスする。エクスプローラを起動し、アドレスバーに`￥￥サーバのIPアドレス`のように入力する。続いて、Samabaユーザのユーザ名とパスワードを入力する。

### ３．ゲスト接続

Windowsからの接続するユーザを全てGuest接続として、すべて許可する。

### ゲストユーザの追加

```shell
[root@localhost ~]# useradd guest
```

### smb.confの設定

```shell
[root@file ~]# vi /etc/samba/smb.conf
```

```conf
# See smb.conf.example for a more detailed config file or
# read the smb.conf manpage.
# Run 'testparm' to verify the config is correct after
# you modified it.

[global]
        workgroup = WORKGROUP
        security = user

        passdb backend = tdbsam

        printing = cups
        printcap name = cups
        load printers = yes
        cups options = raw

        /* 追記部分 */
        map to guest = Bad User
        guest account = guest

[share]
        /* 変更部分 */
        comment = All User Share Directory
        path = /var/samba/share
        browseable = yes
        writable = yes
        guest ok = yes
        create mask = 0644
        directory mask = 0775
```

## ４．ホスト名でのアクセス

### namde.confの設定

以下のコマンドにて、named.confを開きます。

```shell
[root@localhost ~]# vi /etc/named.conf
```

以下の設定箇所を変更

- listen-on port 53 { 127.0.0.1; 10.45.48.0/24; };
- allow-query     { localhost; 10.45.48.0/24; };
- forwarders { 10.45.100.100; };
- forward only;
- zone "j00.sangidai.com" IN {
        type master;
        file "j00.sangidai.zone";
  };

  zone "48.45.10.in-addr.arpa" IN {
        type master;
        file "48.45.10.rzone";
  };

```shell
/
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//
// See the BIND Administrator's Reference Manual (ARM) for details about the
// configuration located in /usr/share/doc/bind-{version}/Bv9ARM.html

options {
        //DNSサーバ及びクライアントPCのネットワークアドレスを追加
        listen-on port 53 { 127.0.0.1; 10.45.48.0/24; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
        //DNSサーバ及びクライアントPCのネットワークアドレスを追加
        allow-query     { localhost; 10.45.48.0/24; };
        forwarders { 10.45.100.100; };

        /* 省略 */
        
        recursion yes;
        forward only;


        dnssec-enable yes;
        dnssec-validation yes;

        /* Path to ISC DLV key */
        bindkeys-file "/etc/named.root.key";

        managed-keys-directory "/var/named/dynamic";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

zone "j00.sangidai.com" IN {
        type master;
        file "j00.sangidai.zone";
};

zone "48.45.10.in-addr.arpa" IN {
        type master;
        file "48.45.10.rzone";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

### named.confのチェック

以下のコマンドで、named.confファイルに設定ミスや記述ミスが無いか確認をします。設定にミスがある場合、デバッグ機能として、エラーコードが吐き出されます。

```shell
[root@localhost ~]# named-checkconf
```

### j00.sangidai.zoneの変更

```shell
[root@localhost ~]# vi /var/named/j00.sangidai.zone
```

```conf
TTL    86400
@       IN      SOA     ns1.j00.sangidai.com. postmaster.j00.sangidai.com. (
                2022112501      ;serial
                3h              ;refresh
                1h              ;retry
                1w              ;expire
                1h )            ;minimum

        IN      NS      ns1.j00.sangidai.com.
        IN      A       10.45.48.45

ns1     IN      A       10.45.48.45
file    IN      CNAME   ns1
```

### 48.45.10.rzoneの変更

```shell
[root@localhost ~]# vi /var/named/48.45.10.rzone
```

```shell
$TTL    86400
@       IN      SOA     ns1.j00.sangidai.com. postmaster.j00.sangidai.com. (
                2022112501      ;serial
                3h              ;refresh
                1h              ;retry
                1w              ;expire
                1h )            ;minimum

        IN      NS      ns1.j00.sangidai.com.

45      IN      PTR     ns1.j00.sangidai.com.
45      IN      PTR     file.j00.sangidai.com.
```

### ゾーンファイルの確認

```shell
[root@localhost ~]# named-checkzone j00.sangidai.com /var/named/j00.sangidai.zone
```

以下のように表示されれば成功

```shell
zone j00.sangidai.com/IN: loaded serial 2022111702
OK
```

```shell
[root@localhost ~]# named-checkzone 48.45.10.in-addr.arpa /var/named/48.45.10.rzone
```

以下のように表示されれば成功

```shell
zone 48.45.10.in-addr.arpa/IN: loaded serial 2022111701
OK
```

### BINDの操作

- BINDの起動

```shell
[root@localhost ~]# systmel start named-chroot.service
```

- OS起動時にBINDの自動起動を有効化

```shell
[root@localhost ~]# systemctl enable named-chroot.service
```

再度、ファイアウォールの設定等を行うこと。

### 動作確認

Windowsからファイルサーバにアクセスする。エクスプローラを起動し、アドレスバーに`￥￥ホスト名`のように入力する。続いて、Samabaユーザのユーザ名とパスワードを入力する。

Windowsのコマンドプロンプトを開き、ファイル共有として接続しているユーザを確認する。

```shell
C:\Users\user>net use
```

```shell
C:\Users\user>net use \\file.j00.sangidai.com\IPC$  /delete
```
