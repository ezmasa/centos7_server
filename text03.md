# サーバー構築実習（CentOS7）

## Webサーバ構築

### Webサーバとは

Webブラウザからのリクエストに応じて静的画面や画像などのホームページのデータをWebブラウザに送ってくれるサーバのこと。ホームページを表示させるために必要なものとなります。

### Webサーバの種類

主なWebサーバソフトウェアについて以下に示す。

- Apache
    - オープンソースであり、高い信頼性と充実した機能を備えていて、世界中で最も多く使われているWebサーバソフトウェアの一つ

- Nginx
    - Apacheと同じオープンソースであり、同時リクエストの処理に特化していて、軽量で処理が早いWebサーバソフトウェア

- Microsoft IIS
    - Microsoft Windowsの標準Webサーバソフトウェア

### Webサーバの導入

#### Apacheのインストール

- httpdのインストール

```shell
[root@localhost ~]# yum -y install httpd
```

以下のように表示されれば成功

```shell
インストール:
  httpd.x86_64 0:2.4.6-97.el7.centos.5                                                                                                                

依存性関連をインストールしました:
  apr.x86_64 0:1.4.8-7.el7      apr-util.x86_64 0:1.5.2-6.el7      httpd-tools.x86_64 0:2.4.6-97.el7.centos.5      mailcap.noarch 0:2.1.41-2.el7     

完了しました!
[root@localhost ~]#
```

- httpdの起動

```shell
[root@localhost ~]# systemctl start httpd.service
```

- httpdの自動起動を有効化

```shell
[root@localhost ~]# systemctl enable httpd.service
```

- httpdの状態確認

```shell
[root@localhost ~]# systemctl status httpd.service
```

Active欄が`active(running)`になっていれば成功

```shell
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since 金 2022-11-18 18:43:26 JST; 24s ago
     Docs: man:httpd(8)
           man:apachectl(8)
```

#### htttpd.confの設定

- httpd.confの中身を確認

```shell
[root@localhost ~]# cat /etc/httpd/conf/httpd.conf 
```

- 以下の箇所を確認
  - `DocumentRootが/var/www/html`になっているか
  - `DirectoryIndex`に`index.html`が入っているか

```shell
/* 省略 */

</Directory>

#
# Note that from this point forward you must specifically allow
# particular features to be enabled - so if something's not working as
# you might expect, make sure that you have specifically enabled it
# below.
#

#
# DocumentRoot: The directory out of which you will serve your
# documents. By default, all requests are taken from this directory, but
# symbolic links and aliases may be used to point to other locations.
#
DocumentRoot "/var/www/html"

#
# Relax access to content within /var/www.
#

/* 省略 */

</Directory>

#
# DirectoryIndex: sets the file that Apache will serve if a directory
# is requested.
#
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

- httpdの再起動

```shell
[root@localhost ~]# systemctl restart httpd.service
```

### DNSサーバの設定
#### j00.sangidai.zoneの変更

CNAMEにて、正規表現のホスト名に対し、別名を付ける

```shell
$TTL    86400
@       IN      SOA     ns1.j00sangidai.com. postmaster.j00.sangidai.com. (
                2022111702      ;serial
                3h              ;refresh
                1h              ;retry
                1w              ;expire
                1h )            ;minimum


        IN      NS      ns1.j00.sangidai.com.

ns1     IN      A       10.45.48.26
www     IN      CNAME   ns1
```

- ゾーンファイルの確認

```shell
[root@localhost ~]# named-checkzone j00.sangidai.com /var/named/j00.sangidai.zone
```

- namedの再起動

```shell
[root@localhost ~]# systemctl restart named-chroot.service
```

#### ファイアウォールの設定

```shell
[root@localhost ~]# firewall-cmd --add-service http
success
```

```shell
[root@localhost ~]# firewall-cmd --add-service http --permanent
success
```

```shell
[root@localhost ~]# firewall-cmd --reload
success
```

### 以下のURLにアクセス

<http://www.j00.sangidai.com>にアクセスする

### HTML言語によるホームページ作成

Webサイトは、HTML言語というマークアップ言語とCSSというスタイルシートによって構成されています。HTMLは、Webブラウザ上に表示する文字などの表示内容を記述する。CSSは、HTMLで書かれた内容に、デザイン性を与える仕組みで、文字の大きさや色、配置など、外観に関する表示内容を記述する言語になっています。

- inde.htmlの作成

```shell
[root@localhost ~]# vi /var/www/html/index.html
```

以下を記述する

```html
<!DOCTYPE html>
<html lang="ja">
<head>
        <title>Hello My Web Server</title>
</head>
<body>
        <h1>Hello Web Server</h1>
        <p>I am Sangiman</p>
</body>
</html>
```

- httpdの再起動

```shell
[root@localhost ~]# systemctl restart httpd.service 
```

### 以下のURLにアクセス

<http://www.j00.sangidai.com>にアクセスする

### CGIによる動的Webサイト作成

HTMLによるホームページの表示は、常に同じ内容のデータがブラウザに送信されます。つまり、同じ内容のみが表示されるということです。昨今のWebサイトでは、表示をするたびに最新の情報が表示されるものになっています。そのためには、サーバ上で、表示するデータを動的に生成し、ブラウザに送信する必要があります。CGI(Common Gateway Interface)は、プログラムによって生成されたデータをWebサーバ経由でブラウザに送信するための、最も基本的な仕組みです。今回は`C言語`を使って、その仕組みを作成します。


#### Webサーバの設定

- httpd.confの変更

```shell
[root@localhost ~]# vi /etc/httpd/conf/httpd.conf 
```

以下の修正箇所を変更する。

- `ExecCGIを追加
  - Options Indexes FollowSymLinks ExecCGI

- #(コメントアウト)を外し、行を有効化
  - AddHandler cgi-script .cgi

```shell
/* 省略 */

</Directory>

# Further relax access to the default document root:
<Directory "/var/www/html">
    #
    # Possible values for the Options directive are "None", "All",
    # or any combination of:
    #   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
    #
    # Note that "MultiViews" must be named *explicitly* --- "Options All"
    # doesn't give it to you.
    #
    # The Options directive is both complicated and important.  Please see
    # http://httpd.apache.org/docs/2.4/mod/core.html#options
    # for more information.
    #
    Options Indexes FollowSymLinks ExecCGI

    #
    # AllowOverride controls what directives may be placed in .htaccess files.
    # It can be "All", "None", or any combination of the keywords:
    #   Options FileInfo AuthConfig Limit
    #

    /* 省略 */

    #
    # AddHandler allows you to map certain file extensions to "handlers":
    # actions unrelated to filetype. These can be either built into the server
    # or added with the Action directive (see below)
    #
    # To use CGI scripts outside of ScriptAliased directories:
    # (You will also need to add "ExecCGI" to the "Options" directive.)
    #
    AddHandler cgi-script .cgi

    # For type maps (negotiated resources):
    #AddHandler type-map var

    #
    # Filters allow you to process content before it is sent to the client.
    #
    # To parse .shtml files for server-side includes (SSI):
    # (You will also need to add "Includes" to the "Options" directive.)
    #
/* 省略 */
```

- httpdの再起動

```shell
[root@localhost ~]# systemctl restart httpd.service 
```

#### プログラムの作成

- プログラムの作成

```shell
[root@localhost ~]# vi /var/www/html/helloCGI.c
```

以下のプログラムを記述する。

```c++
#include <stdio.h>
#include <time.h>

void main(){
        time_t timer;
        struct tm  *local;

        int year,month,day,hour,min,sec;

        timer=time(NULL);
        local=localtime(&timer);

        year=local->tm_year+1900;
        month=local->tm_mon+1;
        day=local->tm_mday;
        hour=local->tm_hour;
        min=local->tm_min;
        sec=local->tm_sec;

        printf("Content-type:text/html\n\n");
        printf("<!DOCTYPE html>\n");
        printf("<head>\n");
        printf("<title>Welcome CGI</title>\n");
        printf("</head>\n");
        printf("<body>\n");
        printf("Hello CGI\n");
        printf("</body>\n");
        printf("<h1>%d:%d:%d_ %d:%d:%d\n</h1>",year,month,day,hour,min,sec);
        printf("<p>I am Sangiman</p>\n");
}
```

#### gccコンパイラのインストール

- gccコンパイラのインストール

```shell
[root@localhost ~]# yum -y install gcc
```

- プログラムファイルのコンパイル

```shell
[root@localhost ~]# gcc -o /var/www/html/helloCGI.cgi /var/www/html/helloCGI.c
```

#### 以下のURLにアクセス

<http://www.j00.sangidai.com/helooCGI.cgi>にアクセスする。

### PHPによる動的Webサイト作成

#### PHPとは

PHPが登場する以前は、CGIスクリプトで動的Webサイトを構築することが主流であったが、PHPは、

#### PHPのインストール

- phpのインストール

```shell
[root@localhost ~]# yum -y install php
```

- phpのバージョン確認

```shell
[root@localhost ~]# php -v
PHP 5.4.16 (cli) (built: Apr  1 2020 04:07:17) 
Copyright (c) 1997-2013 The PHP Group
Zend Engine v2.4.0, Copyright (c) 1998-2013 Zend Technologies
```

#### Webサーバの設定

- Apache(httpd.conf)の変更

```shell
[root@localhost ~]# vi /etc/httpd/conf/httpd.conf 
```

以下の修正箇所を変更する。

- `index.php`を追加
  -　DirectoryIndex index.html index.php

-  `.php`の拡張子をPHPプログラムとして登録
  - AddType application/x-httpd-php .php

```shell
/* 省略 */

</Directory>

#
# DirectoryIndex: sets the file that Apache will serve if a directory
# is requested.
#
<IfModule dir_module>
    DirectoryIndex index.html index.php
</IfModule>

#
# The following lines prevent .htaccess and .htpasswd files from being
# viewed by Web clients.
#
<Files ".ht*">
    Require all denied
</Files>

/* 省略 */

    #AddHandler type-map var

    #
    # Filters allow you to process content before it is sent to the client.
    #
    # To parse .shtml files for server-side includes (SSI):
    # (You will also need to add "Includes" to the "Options" directive.)
    #
    AddType text/html .shtml
    AddType application/x-httpd-php .php
    AddOutputFilter INCLUDES .shtml
</IfModule>

#
# Specify a default charset for all content served; this enables
# interpretation of all content as UTF-8 by default.  To use the
# default browser choice (ISO-8859-1), or to allow the META tags
# in HTML content to override this choice, comment out this
# directive:
#
AddDefaultCharset UTF-8

<IfModule mime_magic_module>
```

#### プログラムの作成

- HTMLファイルの作成

```shell
[root@localhost ~]# vi /var/www/html/input.html
```

以下の内容を記述する。

```html
<!DOCTYPE html>
<head>
        <title>Input Form</title>
</head>
<body>
        <h1>Regist Name</h1>
        <form action="regist.php" method="post">
        <p>Please enter your name</p>
        <p>Given Name:<input type="text" name="given_name"></p>
        <p>Family Name:<input type="text" name="family_name"></p>
        <p><input type="submit" value="Click the button below"></p>
        </form>
<body>
</html>
```

- PHPファイルの作成

```shell
[root@localhost ~]# vi /var/www/html/regist.php
```

以下の内容を記述する。

```php
<!DOCTYPE html>
<head>
        <title>Register Form</title>
</head>
<body>
        <h1>Date Entered</h1>
        <p>
        <?php
        $f_name=$_POST['family_name'];
        $g_name=$_POST['given_name'];

        echo"<p>Gine Name:$g_name</p>";
        echo"<p>Family Name:$f_name</p>";
        ?>
        </p>
</body>
</html>
```

- httpdの再起動

```shell
[root@localhost ~]# systemctl restart httpd.service
```

### 以下のURLにアクセス

<http://www.j00.sangidai.com/input.html>にアクセスする。