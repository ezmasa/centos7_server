# サーバー構築実習（CentOS7）

## DNSサーバ構築

### DNSサーバとは

DNS(ドメイン・ネーム・システム)は、インターネットを利用した階層的な分散データベースを構成する。それぞれのドメインでは、個々にネームサーバが用意され、自ドメイン内のホスト情報をテキスト形式で保持し、リクエストに応答する。そのネームサーバは、上位のドメインを運用するネームサーバに登録され、さらに上位のドメインを運用するネームサーバに登録される。

### DNSサーバの役割

自身の持つデータベースからホストのIPアドレスを提供するだけでなく、クライアントが外部ドメインのホストへ接続する際に、そのホスト（例：www.xxxyyy.co.jp）のIPアドレスを「.」「jp」「co.jp」「xxxyyy.co.jp」と順番にそのドメインを管理するネームサーバに問い合わせてIPアドレスを知り、クライアントへ提供する。誰かがインターネットを利用するとき、必ずこのDNSの階層型分散データベースを利用している。DNSサーバは、インターネットを支える重要な役割を担っている。

## １．簡易的な名前解決

当初、IPアドレスに代わるホストの指定方法として、hostsファイルによる識別が行われていた。当時は、ホストの追加（削除）の度にhostsファイルを更新し、公開していた。

### www.google.comに疎通確認

pingコマンドでホスト名`www.google.com`を指定して、疎通できるのか確認してみる。

```shell
[user@localhost /]$ ping www.google.com
ping: www.google.com: 名前またはサービスが不明です
```

上記のように送れないことが確認できる。

### hostsの権限を確認

ls -l コマンドで、hostsファイルのファイルのパーミッション（所有権）を確認してみる。

```shell
[root@localhost ~]# ls -l /etc/hosts
-rw-r--r--. 1 root root 188 11月 16 17:43 /etc/hosts
```

userには、読込みのみ許可されていることが確認できる。

### userに書込みの権限を付与

chmodコマンドで、userにも書込み可能にする。

```shell
[root@localhost ~]# chmod 666 /etc/hosts
[root@localhost ~]# exit
ログアウト
```

再度、ls -l コマンドで、hostsファイルのファイルのパーミッション（所有権）を確認してみる。

```shell
[root@localhost ~]# ls -l /etc/hosts
-rw-rw-rw-. 1 root root 188 11月 16 17:43 /etc/hosts
```

### hostsファイルの編集

viコマンドでファイルを開く。

```shell
[user@localhost /]$ vi /etc/hosts
```

- 142.251.42.132 www.google.comの追加

```txt
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
142.251.42.132 www.google.com
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```

### 動作確認

再度、pingで疎通確認してみる。

```shell
[user@localhost /]$ ping www.google.com
PING www.google.com (142.251.42.132) 56(84) bytes of data.
64 bytes from www.google.com (142.251.42.132): icmp_seq=1 ttl=57 time=11.1 ms
64 bytes from www.google.com (142.251.42.132): icmp_seq=2 ttl=57 time=16.6 ms
64 bytes from www.google.com (142.251.42.132): icmp_seq=3 ttl=57 time=22.8 ms
```

## ２．DNSキャッシュサーバ構築

### DNSキャッシュサーバとは

DNSキャッシュサーバは、クライアントからの要求を、自身の要求とし、上位のDNSサーバへ問い合わせる。その結果をクライアントに返すのと同時に、その情報をキャッシュするサーバのことである。ネットワーク内で重複するデータを下位のネットワークで処理し、上位のネットワークのトラフィック（ネットワーク上を移動するデータ量）を減らすことができる。

### キャッシュサーバの動作

キャッシュサーバを以下に示す。

- 名前解決を受ける
- 自分のキャッシュデータに該当するものがないかチェック
- 該当したらキャッシュからクライアントへ返し、処理する。該当しない場合は、上位DNSへ問い合わせる
- 応答からデータをキャッシュに加えると同時にクライアントへ返す

### DNSサーバソフト（BIND）のインストール

- BINDのインストール

```shell
[root@localhost ~]# yum -y install bind
```

以下のように表示されれば成功です。

```shell
Loading mirror speeds from cached hostfile
 * base: ftp-srv2.kddilabs.jp
 * extras: ftp-srv2.kddilabs.jp
 * updates: ftp-srv2.kddilabs.jp
依存性の解決をしています
--> トランザクションの確認を実行しています。
---> パッケージ bind.x86_64 32:9.11.4-26.P2.el7_9.10 を インストール

完了しました!
```

- bind-chrootのインストール

サーバ自身のセキュリティ強度向上のため、bind-chrootを導入

```shell
[root@localhost ~]# yum -y install bind-chroot
```

以下のように表示されれば成功です。

```shell
--> トランザクションの確認を実行しています。
---> パッケージ bind-chroot.x86_64 32:9.11.4-26.P2.el7_9.10 を インストール
--> 依存性解決を終了しました。

インストール:
  bind-chroot.x86_64 32:9.11.4-26.P2.el7_9.10                                   

完了しました!
```

- bind-utilsのインストール

設定ファイルや動作チェック用の各種ツール

```shell
[root@localhost ~]# yum -y install bind-utils
```

デフォルトで導入されている場合、以下のように表示されます。

```shell
読み込んだプラグイン:fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: ftp-srv2.kddilabs.jp
 * extras: ftp-srv2.kddilabs.jp
 * updates: ftp-srv2.kddilabs.jp
パッケージ 32:bind-utils-9.11.4-26.P2.el7_9.10.x86_64 はインストール済みか最新バージョンです
何もしません
```

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

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

### named.confのチェック

以下のコマンドで、named.confファイルに設定ミスや記述ミスが無いか確認をします。設定にミスがある場合、デバッグ機能として、エラーコードが吐き出されます。

```shell
[root@localhost ~]# named-checkconf
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

- BINDのステータスを表示

以下のようにActive(running)と表示されれば成功です。

```shell
[root@localhost ~]# systemctl status named-chroot.service
● named-chroot.service - Berkeley Internet Name Domain (DNS)
   Loaded: loaded (/usr/lib/systemd/system/named-chroot.service; enabled; vendor preset: disabled)
   Active: active (running) since 木 2022-11-17 17:57:04 JST; 23min ago
  Process: 1110 ExecStart=/usr/sbin/named -u named -c ${NAMEDCONF} -t /var/named/chroot $OPTIONS (code=exited, status=0/SUCCESS)
  Process: 1072 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == "yes" ]; then /usr/sbin/named-checkconf -t /var/named/chroot -z "$NAMEDCONF"; else echo "Checking of zone files is disabled"; fi (code=exited, status=0/SUCCESS)
 Main PID: 1122 (named)
   CGroup: /system.slice/named-chroot.service
           └─1122 /usr/sbin/named -u named -c /etc/named.conf -t /var/named/chroot
```

### ファイアウォールの操作

- FirewallにDNSサービス用の接続を許可

```shell
[root@localhost ~]# firewall-cmd --add-service=dns --zone=public
success
```

- 恒久的に適用

```shell
[root@localhost ~]# firewall-cmd --add-service=dns --zone=public --permanent
success
```

- Firewallを再起動

```shell
[root@localhost ~]# firewall-cmd --reload
success
```

- FirewallにDNSが許可されているかを確認

```shell
[root@localhost ~]# firewall-cmd --list-service
ssh dhcpv6-client dns
```

- DNSのポート（53番ポート）の状態確認

DNSサーバ(Centos)のIPアドレスに53がくっついている

```shell
[root@localhost ~]# netstat -nau
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
udp        0      0 10.45.48.26:53          0.0.0.0:*  
```

### 動作確認

クライアントPCにて、DNSサーバのIPアドレスを、DNSサーバとして、指定する。

```shell
C:\Users\user>ping www.google.com
ww.google.com [142.250.196.100]に ping を送信しています 32 バイトのデータ:                                             142.250.196.100 からの応答: バイト数 =32 時間 =8ms TTL=57                                          142.250.196.100 からの応答: バイト数 =32 時間 =9ms TTL=57                                          142.250.196.100 からの応答: バイト数 =32 時間 =9ms TTL=57                                          142.250.196.100 からの応答: バイト数 =32 時間 =9ms TTL=57               142.250.196.100 の ping 統計:                                               パケット数: 送信 = 4、受信 = 4、損失 = 0 (0% の損失)、                                           ラウンド トリップの概算時間 (ミリ秒):                                               最小 = 8ms、最大 = 9ms、平均 = 8ms
```

## ３．DNSネームサーバ構築

### DNSネームサーバとは

DNSサーバには、「キャッシュサーバ（フルリゾルバ）」のほかに、「ネームサーバ（権威サーバ、コンテンツDNS）」があります。ネームサーバは、自身の管理するドメインに所属するホストの名前に対応するIPアドレスを提供します。

### 構築するネームサーバの設定

|  名前  |  ホスト名 |  IPアドレス  |
| ---- | ---- | ---- |
|  ネームサーバ  |  ns1.j00.sangidai.com  |　10.45.46.26 |
|  管理者メアド  |  postmaster@j00.sangidai.com  |  10.45.46.26  |


### named.confの設定

/* 追記部分 */以下を追加する。

```shell
zone "." IN {
        type hint;
        file "named.ca";
};

/* 追記部分 */

zone "j00.sangidai.com" IN {
        type master;
        file "j00.sangidai.zone";
};

zone "48.45.10.in-addr.arpa" IN {
        type master;
        file "48.45.10.rzone";
};
```

### named.confのチェック

以下のコマンドで、named.confファイルに設定ミスや記述ミスが無いか確認をします。設定にミスがある場合、デバッグ機能として、エラーコードが吐き出されます。

```shell
[root@localhost ~]# named-checkconf
```

### ゾーンファイルとは

- 正引き：ホスト名からIPアドレスを知ること。
- 逆引き：IPアドレスからホスト名を知ること。

ファイルの保存場所と名前は、`named.conf`ファイル上の記述を参照し、新規に作成する。

- 作成するゾーン
  - 正引きドメイン「j00.sangidai.com」
  - 逆引きドメイン「46.45.10.in-addr.arpa」

### ゾーンファイルの作成

- j00.sangidai.zoneを作成

```shell
[root@localhost ~]# vi /var/named/j00.sangidai.zone
```

以下のように記述する。

```shell
$TTL    86400
@       IN      SOA     ns1.j00sangidai.com. postmaster.j00.sangidai.com. (
                2022111702      ;serial
                3h              ;refresh
                1h              ;retry
                1w              ;expire
                1h )            ;minimum


        IN      NS      ns1.j00.sangidai.com.
        IN      A       10.46.48.26

ns1     IN      A       10.45.48.26
```

- 46.45.10.rzoneを作成

```shell
[root@localhost ~]# vi /var/named/48.45.10.rzone
```

以下のように記述する。

```shell
$TTL    86400
@       IN      SOA     ns1.j00.sangidai.com. postmaster.j00.sangidai.com. (
                2022111701      ;serial
                3h              ;refresh
                1h              ;retry
                1w              ;expire
                1h )            ;minimum

        IN      NS      ns1.j00.sangidai.com.
26      IN      PTR     ns1.j00.sangidai.com.
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

### BINDサービスの再起動

```shell
[root@localhost ~]# systemctl restart named-chroot.service
```

再度、ファイアウォールの設定等を行うこと。

### 動作確認

