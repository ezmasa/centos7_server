# サーバー構築実習（CentOS7）

## SSHサーバ構築

## １．SSHサーバ（OpenSSH）の導入

### opensshのインストール

CentOSでは、標準でSSHサーバ`OpenSSH`が導入されており、自動的に起動するようになっている。以下のコマンドを使って、SSHサーバの導入に必要な３つのパッケージがインストールされているか確認する。

```shell
[root@localhost ~]# yum list installed | grep openssh
libssh2.x86_64                             1.4.3-10.el7_2.1            @anaconda
openssh.x86_64                             7.4p1-11.el7                @anaconda
openssh-clients.x86_64                     7.4p1-11.el7                @anaconda
openssh-server.x86_64                      7.4p1-11.el7                @anaconda
```

- アップデート可能なパッケージを確認するコマンド

```shell
[root@localhost ~]# yum list updates
```

SSHサーバに必要なパッケージのアップデートを行う。

```shell
[root@localhost ~]# yum -y update openssh openssh-server openssh-clients
```

### sshd_configの設定

```shell
[root@localhost ~]# vim /etc/ssh/sshd_config 
```

- 38行目
  - #PermitRootLogin yes
  - PermitRootLogin no

```conf
/* 省略 */

# Logging
#SyslogFacility AUTH
SyslogFacility AUTHPRIV
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
#PermitRootLogin yes
PermitRootLogin no
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

### SSHサービスの起動

```shell
[root@localhost ~]# systemctl start sshd.service
```

### SSHサービスの自動起動の有効化

```shell
[root@localhost ~]# systemctl enable sshd.service
```

### SSHサービスの状態確認

以下のように`Active: active(running)`になっていれば成功

```shell
[root@localhost ~]# systemctl status sshd.service
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
   Active: active (running) since 木 2023-01-05 13:19:32 JST; 5min ago
     Docs: man:sshd(8)
           man:sshd_config(5)
 Main PID: 14345 (sshd)
   CGroup: /system.slice/sshd.service
           └─14345 /usr/sbin/sshd -D

 1月 05 13:19:32 localhost.localdomain systemd[1]: Starting OpenSSH server d...
 1月 05 13:19:32 localhost.localdomain sshd[14345]: Server listening on 0.0....
 1月 05 13:19:32 localhost.localdomain sshd[14345]: Server listening on :: p...
 1月 05 13:19:32 localhost.localdomain systemd[1]: Started OpenSSH server da...
Hint: Some lines were ellipsized, use -l to show in full.
```

### Linux端末の接続先IPアドレスの設定

```shell
[root@localhost ~]# ssh 10.45.48.14
The authenticity of host '10.45.48.14 (10.45.48.14)' can't be established.
ECDSA key fingerprint is SHA256:lWlMJ4ML43rKAcA8xseHtVtlF8za5Z3Va0Z+CMPNoMM.
ECDSA key fingerprint is MD5:92:2b:92:10:ba:13:bc:88:5c:59:45:c0:7d:c8:d0:b8.
Are you sure you want to continue connecting (yes/no)? yes
```

### 接続先パスワードの設定

混乱するのでuserpassに設定する。

```shell
root@10.45.48.14's password: 
Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
```

### 動作確認



## ２．鍵交換方式による認証

一般的なパスワード方式の認証では、パスワードを盗まれる（見られる）と、多くの権限を奪われる（利用される）などといったことになります。ここでは、ユーザとしてのパスワードを利用しない、別のパスフレーズと公開鍵/秘密鍵を使ったSSH接続の方法を行います。

- 鍵交換方式による認証
  - 公開鍵
    - 接続先となるサーバPCに保存する
  - 秘密鍵
    - ユーザが直接操作するクライアントPCに保存する


```shell
[user@localhost ~]$ mkdir .ssh
```

```shell
[user@localhost ~]$ chmod 600 .ssh
```

```shell
[user@localhost ~]$ mv id_rsa.pub .ssh/authorized_keys
```

