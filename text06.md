# サーバー構築実習（CentOS7）

## メールサーバ構築

### メールとは

電子メールは、仕事だけではなくプライベートな生活でも欠かせないツールです。しかし、電子メールには危険な側面が多く存在し、適正な運用を求められる。

## １．DNSサーバの導入

### DNSサーバの設定

VM Wareを2つ同時に起動し、サーバPCを2台立ち上げる。
1つをMailサーバ(IP:10.45.46.yy)、もう1つをDNSサーバ(IP:10.45.46.xx)とする。

### 構築するサーバの設定

|  名前  |  ホスト名 |  ドメイン名  |  IPアドレス  |
| ---- | ---- | ---- | ---- |
|  ネームサーバ  |  dns1  |　itxx.sangi.com  |  10.45.46.xx |
|  管理者メアド  |  postmaster  |  itxx.sangi.com  |  -  |
| メールサーバ  |  mail  |  itxx.sangi.com  |  10.45.46.yy  |

### namde.confの設定

以下のコマンドにて、named.confを開きます。

```shell
[root@localhost ~]# vim /etc/named.conf
```

以下の設定箇所を変更

- listen-on port 53 { 127.0.0.1; 10.45.46.0/24; };
- allow-query     { localhost; 10.45.46.0/24; };
- forwarders { 10.45.100.100; };
- forward only;
- 正引きゾーンと逆引きゾーンの追加

| ゾーン種類 | タイプ | ゾーン名 | ファイル名 |
| --- | --- | --- | --- |
| 正引き | master |itxx.sangi.com | itxx.sangi.com.zone |
| 逆引き | master | 46.45.10.in-addr.arpa | 46.45.10.in-addr.arpa.rzone |

### named.confのチェック

以下のコマンドで、named.confファイルに設定ミスや記述ミスが無いか確認をします。設定にミスがある場合、デバッグ機能として、エラーコードが吐き出されます。

```shell
[root@localhost ~]# named-checkconf
```

### itxx.sangi.com.zoneの変更

```shell
[root@localhost ~]# vim /var/named/itxx.sangi.com.zone
```

- Aレコードの追記

```shell
$TTL    86400

/* 省略 */

dns1     IN      A       10.45.48.xx
mail    IN      A       10.45.48.yy
```

### 46.45.10.in-addr.arpa.rzoneの変更

```shell
[root@localhost ~]# vim /var/named/46.45.10.in-addr.arpa.rzone
```

- PTRレコードの追加


```shell
$TTL    86400

/* 省略 */

xx      IN      PTR     dns1.itxx.sangi.com.
yy      IN      PTR     mail.jitxx.sangi.com.
```

## ２．メールサーバ（Postfix, Dovecot）の導入

代表的なLinux用メールサーバソフト

- SMTPサーバ「Postfix」
  - ユーザが送信に利用する。または、ドメイン間のメール配送に使用される。

- POP3/IMAPサーバ「Dovecot」
  - ユーザが自分宛のメールを確認する際に使用される。

### PostfixとDovecotのインストール

```shell
[root@localhost ~]# yum -y install postfix
```

```shell
[root@localhost ~]# yum -y install dovecot
```

### ファイアウォールの設定

```shell
[root@localhost ~]# firewall-cmd --add-service=smtp --zone=public --permanent
```

```shell
[root@localhost ~]# firewall-cmd --add-port=110/tcp --zone=public --permanent
```

```shell
[root@localhost ~]# firewall-cmd --add-port=143/tcp --zone=public --permanent
```

```shell
[root@localhost ~]# firewall-cmd --reload
```

```shell
[root@localhost ~]# vim /etc/postfix/main.cf 
```

### main.cfの設定

以下のコマンドにて、main.cfを開きます。

```shell
[root@localhost ~]# vi /etc/postfix/main.cf
```

以下の設定箇所を変更する。

- 76行目
  - myhostname = mail.itxx.sangi.com
- 83行目
  - mydomain = itxx.sangi.com
- 99行目
  - myorigin = $mydomain
- 113行目
  - inet_interfaces = all
- 116行目
  - #inet_interfaces = localhost
- 164行目
  - #mydestination = $myhostname, localhost,.$mydomain, localhost
- 165行目
  - mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
- 208行目
  - local_recipient_maps = unix:passwd.byname $alias_maps
- 264行目
  - mynetworks = 127.0.0.1, 10.45.46.0/24
- 419行目
  - home_maikbix = Maildir/


```conf

/* 省略 */

# INTERNET HOST AND DOMAIN NAMES
# 
# The myhostname parameter specifies the internet hostname of this
# mail system. The default is to use the fully-qualified domain name
# from gethostname(). $myhostname is used as a default value for many
# other configuration parameters.
#
#myhostname = host.domain.tld
#myhostname = virtual.domain.tld
myhostname = mail.itxx.sangi.com
# The mydomain parameter specifies the local internet domain name.
# The default is to use $myhostname minus the first component.
# $mydomain is used as a default value for many other configuration
# parameters.
#
#mydomain = domain.tld
mydomain = itxx.sangi.com

# SENDING MAIL
# 
# The myorigin parameter specifies the domain that locally-posted
# mail appears to come from. The default is to append $myhostname,
# which is fine for small sites.  If you run a domain with multiple
# machines, you should (1) change this to $mydomain and (2) set up
# a domain-wide alias database that aliases each user to
# user@that.users.mailhost.
#
# For the sake of consistency between sender and recipient addresses,
# myorigin also specifies the default domain name that is appended
# to recipient addresses that have no @domain part.
#
#myorigin = $myhostname
myorigin = $mydomain

/* 省略  */

# Specify a list of host or domain names, /file/name or type:table
# patterns, separated by commas and/or whitespace. A /file/name
# pattern is replaced by its contents; a type:table is matched when
# a name matches a lookup key (the right-hand side is ignored).
# Continue long lines by starting the next line with whitespace.
#
# See also below, section "REJECTING MAIL FOR UNKNOWN LOCAL USERS".
#
#mydestination = $myhostname, localhost.$mydomain, localhost
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
#mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain,
#       mail.$mydomain, www.$mydomain, ftp.$mydomain

/* 省略 */

# The right-hand side of the lookup tables is conveniently ignored.
# In the left-hand side, specify a bare username, an @domain.tld
# wild-card, or specify a user@domain.tld address.
# 
local_recipient_maps = unix:passwd.byname $alias_maps
#local_recipient_maps = proxy:unix:passwd.byname $alias_maps
#local_recipient_maps =

# The unknown_local_recipient_reject_code specifies the SMTP server
# response code when a recipient domain matches $mydestination or
# ${proxy,inet}_interfaces, while $local_recipient_maps is non-empty
# and the recipient address or address local-part is not found.
#
# The default setting is 550 (reject mail) but it is safer to start
# with 450 (try again later) until you are certain that your
# local_recipient_maps settings are OK.
#
unknown_local_recipient_reject_code = 550

/* 省略 */

# You can also specify the absolute pathname of a pattern file instead
# of listing the patterns here. Specify type:table for table-based lookups
# (the value on the table right-hand side is not used).
#
#mynetworks = 168.100.189.0/28, 127.0.0.0/8
mynetworks = 127.0.0.1, 10.45.46.0/24
#mynetworks = $config_directory/mynetworks
#mynetworks = hash:/etc/postfix/network_table

# The relay_domains parameter restricts what destinations this system will
# relay mail to.  See the smtpd_recipient_restrictions description in
# postconf(5) for detailed information.

/* 省略 */

# DELIVERY TO MAILBOX
#
# The home_mailbox parameter specifies the optional pathname of a
# mailbox file relative to a user's home directory. The default
# mailbox file is /var/spool/mail/user or /var/mail/user.  Specify
# "Maildir/" for qmail-style delivery (the / is required).
#
#home_mailbox = Mailbox
home_mailbox = Maildir/

# The mail_spool_directory parameter specifies the directory where
# UNIX-style mailboxes are kept. The default setting depends on the
# system type.
#
#mail_spool_directory = /var/mail
#mail_spool_directory = /var/spool/mail
```

### main.cfのチェック

```shell
[root@localhost ~]# postfix check
```

### dovecot.confの設定

以下のコマンドにて、dovecot.confを開きます。

```shell
[root@localhost ~]# vim /etc/dovecot/dovecot.conf 
```

以下の設定箇所を変更する。

- 24行目
  - protocols = imap pop3 lmtp

```conf

/* 省略 */

# Default values are shown for each setting, it's not required to uncomment
# those. These are exceptions to this though: No sections (e.g. namespace {})
# or plugin settings are added by default, they're listed only as examples.
# Paths are also just examples with the real defaults being based on configure
# options. The paths listed here are for configure --prefix=/usr
# --sysconfdir=/etc --localstatedir=/var

# Protocols we want to be serving.
protocols = imap pop3 lmtp

# A comma separated list of IPs or hosts where to listen in for connections. 
# "*" listens in all IPv4 interfaces, "::" listens in all IPv6 interfaces.
# If you want to specify non-default ports or anything more complex,
# edit conf.d/master.conf.
#listen = *, ::
```

### 10-mail.confの設定

以下のコマンドにて、10-mail.confを開きます。

```shell
[root@localhost ~]# vim /etc/dovecot/conf.d/10-mail.conf 
```

以下の設定箇所を変更する。

- 30行目
  - mail_location = mail:~/Maildir
- 196行目
  - valid_chroot_dirs = /home

```conf

/* 省略 */

# See doc/wiki/Variables.txt for full list. Some examples:
#
#   mail_location = maildir:~/Maildir
#   mail_location = mbox:~/mail:INBOX=/var/mail/%u
#   mail_location = mbox:/var/mail/%d/%1n/%n:INDEX=/var/indexes/%d/%1n/%n
#
# <doc/wiki/MailLocation.txt>
#
mail_location = maildir:~/Maildir

/* 省略 */

# ':' separated list of directories under which chrooting is allowed for mail
# processes (ie. /var/mail will allow chrooting to /var/mail/foo/bar too).
# This setting doesn't affect login_chroot, mail_chroot or auth chroot
# settings. If this setting is empty, "/./" in home dirs are ignored.
# WARNING: Never add directories here which local users can modify, that
# may lead to root exploit. Usually this should be done only if you don't
# allow shell access for users. <doc/wiki/Chrooting.txt>
valid_chroot_dirs = /home
```

### 10-auth.confの設定

以下のコマンドにて、10-auth.confを開きます。

```shell
[root@localhost ~]# vim /etc/dovecot/conf.d/10-auth.conf 
```

以下の設定箇所を変更する。

- 10行目
  - disable_plaintext_auth = no
- 100行目
  - auth_mechanisms = plain login

```conf

/* 省略 */

## Authentication processes
##

# Disable LOGIN command and all other plaintext authentications unless
# SSL/TLS is used (LOGINDISABLED capability). Note that if the remote IP
# matches the local IP (ie. you're connecting from the same computer), the
# connection is considered secure and plaintext authentication is allowed.
# See also ssl=required setting.
disable_plaintext_auth = no

/* 省略 */

# Take the username from client's SSL certificate, using 
# X509_NAME_get_text_by_NID() which returns the subject's DN's
# CommonName. 
#auth_ssl_username_from_cert = no

# Space separated list of wanted authentication mechanisms:
#   plain login digest-md5 cram-md5 ntlm rpa apop anonymous gssapi otp skey
#   gss-spnego
# NOTE: See also disable_plaintext_auth setting.
auth_mechanisms = plain login

##
## Password and user databases
##
```

### 10-master.confの設定

以下のコマンドにて、10-master.confを開きます。

```shell
[root@localhost ~]# vim /etc/dovecot/conf.d/10-master.conf 
```

以下の設定箇所を変更する。

- 19行目
  - port = 143
- 40行目
  - port = 110

```conf

/* 省略 */

# Internal user is used by unprivileged processes. It should be separate from
# login user, so that login processes can't disturb other processes.
#default_internal_user = dovecot

service imap-login {
  inet_listener imap {
    port = 143
  }
  inet_listener imaps {
    #port = 993
    #ssl = yes
  }

/* 省略 */

service pop3-login {
  inet_listener pop3 {
    port = 110
  }
  inet_listener pop3s {
    #port = 995
    #ssl = yes
  }
}

service lmtp {
  unix_listener lmtp {
    #mode = 0666
  }
```

### 10-ssl.confの設定

以下のコマンドにて、10-ssl.confを開きます。

```shell
[root@localhost ~]# vim /etc/dovecot/conf.d/10-ssl.conf 
```

以下の設定箇所を変更する。

- 8行目
  - ssl = no

```conf
##
## SSL settings
##

# SSL/TLS support: yes, no, required. <doc/wiki/SSL.txt>
# disable plain pop3 and imap, allowed are only pop3+TLS, pop3s, imap+TLS and imaps
# plain imap and pop3 are still allowed for local connections
ssl = no

# PEM encoded X.509 SSL/TLS certificate and private key. They're opened before
# dropping root privileges, so keep the key file unreadable by anyone but
# root. Included doc/mkcert.sh can be used to easily generate self-signed
# certificate, just make sure to update the domains in dovecot-openssl.cnf
ssl_cert = </etc/pki/dovecot/certs/dovecot.pem
ssl_key = </etc/pki/dovecot/private/dovecot.pem
```

### メールボックスの作成

- 既存ユーザにメールボックスを作成

```shell
[root@localhost ~]# mkdir -p /etc/skel/Maildir/{new,cur,tmp}
```

```shell
[root@localhost ~]# chmod -R 700 /etc/skel/Maildir/
```

- 新規ユーザにメールボックスを作成

```shell
[user@localhost ~]$ mkdir ~/Maildir
```

```shell
[user@localhost ~]$ chmod -R 700 ~/Maildir/
```

### PostfixとDovecotサービスの起動

```shell
[root@localhost ~]# systemctl start postfix.service
```

```shell
[root@localhost ~]# systemctl start dovecot.service
```

### PostfixとDovecotサービスの自動起動の有効化

```shell
[root@localhost ~]# systemctl enable postfix.service
```

```shell
[root@localhost ~]# systemctl enable dovecot.service
```

## 動作確認

### 簡易動作チェック

自分から自身にメールを送信し、受信する

- メール送信（Ctrl + Dで送信）

```shell
[user@localhost ~ ]$ mail user@itxx.sangi.com
Subuject: こんにちは
お元気ですか
```

- メール受信

```shell
[user@localhost ~ ]$mail -f ~/Maildir
```

受信したメールリストが表示されるので、番号を打つと、閲覧可能になる。

### Thunderbirdによるチェック

1. 教科書「CentOS7 サーバー徹底構築」のp.308見ながら、CentOSにThunderbirdをインストールする。

2. userでメールアカウントにログインする。

3. 新規ユーザを追加する。

4. Windows10にもThunderbirdをインストールする。

5. 新規ユーザでメールアカウントにログインする。

6. 相互間でメールの送受信を行う。
