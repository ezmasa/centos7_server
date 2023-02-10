# サーバー構築実習（CentOS7）

## SSHサーバ構築

### SSHとは

コンピュータネットワークの通信プロトコルには、多くの種類がある。リモートアクセスとは、ネットワー越しにコンピュータを遠隔操作することをいう。以前は、遠隔でコンピュータを操作することをCUI・GUI関係なくリモートアクセスと総称して呼ばれていた。最近では、GUIで操作することをリモートデスクトップと呼び、差別化している。リモートアクセスには、いくつか種類があり、その一つにSShがある。

| 種類 | 特徴 |
| --- | --- |
| telnet | CUI/暗号化なし |
| SSH | CUI/暗号化通信 |
| RDP | GUI/Winwdow |
| VNS | GUI/各種OS対応 |

## １．SSHサーバ（OpenSSH）の導入

### opensshのインストール

CentOS7では、標準でSSHサーバ`OpenSSH`が導入されており、自動的に起動するようになっている。以下のコマンドを使って、SSHサーバの導入に必要な３つのパッケージがインストールされているか確認する。

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

- SSHサーバに必要なパッケージのアップデート

```shell
[root@localhost ~]# yum -y update openssh openssh-server openssh-clients
```

### sshd_configの設定

- rootのリモートログインを制限（不許可）する

```shell
[root@localhost ~]# vim /etc/ssh/sshd_config 
```

以下の内容を変更する。

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

### ファイアウォールの確認

- sshが許可されていることを確認

```shell
[root@localhost ~]# firewall-cmd --list-service
```

### Linux端末の接続先IPアドレスの設定

- CentOSのIPアドレスを設定する

- Are you sure you want to continue connecting (yes/no)?
  - yes

```shell
[root@localhost ~]# ssh 10.45.46.xx

The authenticity of host '10.45.48.14 (10.45.48.14)' can't be established.
ECDSA key fingerprint is SHA256:lWlMJ4ML43rKAcA8xseHtVtlF8za5Z3Va0Z+CMPNoMM.
ECDSA key fingerprint is MD5:92:2b:92:10:ba:13:bc:88:5c:59:45:c0:7d:c8:d0:b8.
Are you sure you want to continue connecting (yes/no)? yes
```

### 接続先パスワードの設定

混乱するのでuserpassに設定する。

```shell
root@10.45.46.xx's password: 
Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
```

### 動作確認

Tera Termを用いてWindowsからリモートアクセスする。

1. TCP/IPを選択
2. ホストを接続先のIPアドレスに設定
3. サービスをSSHに選択

| 設定項目 | 内容 |
| --- | --- |
| ユーザ名 | user
| パスフレーズ | userpass |
| 認証方式 | ブレインパスワード |

## ２．鍵交換方式による認証

一般的なパスワード方式の認証では、パスワードを盗まれる（見られる）と、多くの権限を奪われる（利用される）などといったことになります。ここでは、ユーザとしてのパスワードを利用しない、別のパスフレーズと公開鍵/秘密鍵を使ったSSH接続の方法を行います。

- 鍵交換方式による認証
  - 公開鍵
    - 接続先となるサーバPCに保存する
  - 秘密鍵
    - ユーザが直接操作するクライアントPCに保存する

### SSH鍵の生成

教科書「CentOS7 サーバー徹底構築」のp.272見ながら、WindowsのTera Termで公開鍵と秘密鍵を作成する。

- パスフレーズ：`sangidai`

※いずれも、保管場所と名前を忘れないようにする。秘密鍵は重要は鍵です。他社がアクセスできないようなフォルダに保存すること。教科書「CentOS7 サーバー徹底構築」のp.273：秘密鍵の保存先
### ファイルの転送

作成した公開鍵のファイルをSSHサーバへ転送する。

- 隠しフォルダ`「.ssh」`を作成

```shell
[user@localhost ~]$ mkdir .ssh
```

- 鍵を保存したフォルダから`Tera Term`の画面内にドラッグ&ドロップする。

- ファイルの転送：SCPを選択
- 送信先（T）：　`.ssh/`

- パーミッションの変更
 - 600 : 所有者のみ読み書き可能

```shell
[user@localhost ~]$ chmod 600 .ssh/id_rsa.pub
```

- ファイル名の変更

```shell
[user@localhost ~]$ mv .ssh/id_rsa.pub .ssh/authorized_keys
```

### 動作確認

Tera Termから新たな接続を行い、リモートアクセスする。

| 設定項目 | 内容 |
| --- | --- |
| ユーザ名 | user
| パスフレーズ | userpass |
| 認証方式 | RSA/DSA/ECDSA/ED25519鍵を使う |

秘密鍵をクリックし、ファイル`id_rsa`を指定する。
