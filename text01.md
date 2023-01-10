# サーバー構築実習（CentOS7）

## Linux基本コマンド

### 管理者（root）と一般ユーザのコマンド

#### ユーザ切り替えコマンド

- su -
    - 管理者への切り替えをする

```shell
[user@localhost ~]$ su -
パスワード:
[root@localhost ~]# 
```

- exit
    - 一般ユーザに戻る

```shell
[root@localhost ~]# exit
ログアウト
[user@localhost ~]$ 
```

#### ユーザを追加するコマンド

- useradd ユーザ名
    - ユーザを追加する

```shell
[root@localhost ~]# useradd hirota
```

- passwd ユーザ名
    - ユーザのパスワードを指定する

```shell
[root@localhost ~]# passwd hirota
ユーザー hirota のパスワードを変更。
新しいパスワード:
新しいパスワードを再入力してください:
passwd: すべての認証トークンが正しく更新できました。
[root@localhost ~]#
```

#### ユーザを削除するコマンド

- userdel ユーザ名
    - ユーザを削除する

```shell
[root@localhost ~]# userdel hirota
```

- rm -rf /home/ユーザ名
    - ユーザのホームディレクトリを削除する

```shell
[root@localhost ~]# rm -rf /home/hirota
```

### ディレクトリとファイルの確認・移動に使うコマンド

#### 現在の作業ディレクトリの絶対パスを表示するコマンド

- pwd
    - 現在どのディレクトリにいるのか確認する

```shell
[user@localhost ~]$ pwd
/home/user
```

#### 作業ディレクトリを変更するコマンド

- cd
    - ホームディレクトリに移動する。

```shell
[user@localhost ~]$ pwd
/home/user
```

- cd ../
    - 一個前の親ディレクトリに移動する。

```shell
[user@localhost ~]$ cd ../
[user@localhost home]$ pwd
/home
[user@localhost home]$ cd ../
[user@localhost /]$ pwd
/
```

- cd /ディレクトリ名
    - 指定したディレクトリに移動する。

```shell
[user@localhost /]$ cd /home/user/
[user@localhost ~]$ pwd
/home/user
```

#### ディレクトリの内容をリスト表示するコマンド

- ls
    - 現在のディレクトリに含まれるファイル名を表示する。

```shell
[user@localhost ~]$ ls
ダウンロード  デスクトップ  ビデオ  画像
テンプレート  ドキュメント  音楽    公開
```

- ls -l
    - パーミッションや所有者等の関連情報を付加して表示する。

```shell
[user@localhost ~]$ ls -l
合計 0
drwxr-xr-x. 2 user user 6 11月 16 15:08 ダウンロード
drwxr-xr-x. 2 user user 6 11月 16 15:08 テンプレート
drwxr-xr-x. 2 user user 6 11月 16 15:08 デスクトップ
drwxr-xr-x. 2 user user 6 11月 16 15:08 ドキュメント
drwxr-xr-x. 2 user user 6 11月 16 15:08 ビデオ
drwxr-xr-x. 2 user user 6 11月 16 15:08 音楽
drwxr-xr-x. 2 user user 6 11月 16 15:08 画像
drwxr-xr-x. 2 user user 6 11月 16 15:08 公開
```

- ls -a
    - 隠しファイル・隠しフォルダを含めて表示する。

```shell
[user@localhost ~]$ ls -a
.              .bash_profile  .esd_auth     テンプレート  音楽
..             .bashrc        .local        デスクトップ  画像
.ICEauthority  .cache         .mozilla      ドキュメント  公開
.bash_logout   .config        ダウンロード  ビデオ
```

### ディレクトリとファイルの操作で使うコマンド

#### ディレクトリを作成するコマンド

- mkdir ディレクトリ名
    - 指定したパスの途中のディレクトリが存在しない場合はエラーになる。

```shell
[user@localhost ~]$ mkdir sampledir
[user@localhost ~]$ ls
sampledir     テンプレート  ドキュメント  音楽  公開
ダウンロード  デスクトップ  ビデオ        画像
```

- mkdir -p ディレクトリ名
    - 途中のディレクトリが存在しない場合、それらを含めてディレクトリを作成することができる。

#### ファイルを作成するコマンド

- vi ファイル名
    - 指定したファイルを作成する。

```shell
[user@localhost ~]$ vi sample1.txt
[user@localhost ~]$ ls
sample1.txt  ダウンロード  デスクトップ  ビデオ  画像
sampledir    テンプレート  ドキュメント  音楽    公開
```

#### ファイルをコピーするコマンド

- cp ファイル名A ファイル名B
    - ファイルAを別名のファイルBにコピーする。

```shell
[user@localhost ~]$ cp sample1.txt sample2.txt
[user@localhost ~]$ ls
sample1.txt  sampledir     テンプレート  ドキュメント  音楽  公開
sample2.txt  ダウンロード  デスクトップ  ビデオ        画像
```

- cp ファイル名 ディレクトリ名
    - ファイルを同名でディレクトリにコピーする。

```shell
[user@localhost ~]$ cp sample2.txt sampledir
[user@localhost ~]$ cd sampledir/
[user@localhost sampledir]$ ls
sample2.txt
```

#### ファイルの移動及びリネームを行うコマンド

- mv ファイル名A ファイル名B
    - ファイルAをファイルBに名前を変更します。

```shell
[user@localhost ~]$ mv sample2.txt sample3.txt
[user@localhost ~]$ ls
sample1.txt  sampledir     テンプレート  ドキュメント  音楽  公開
sample3.txt  ダウンロード  デスクトップ  ビデオ        画像
```

- mv ファイル ディレクトリ
    - ファイルをディレクトリに移動する。

```shell
[user@localhost ~]$ mv sample3.txt sampledir
[user@localhost ~]$ ls
sample1.txt  ダウンロード  デスクトップ  ビデオ  画像
sampledir    テンプレート  ドキュメント  音楽    公開
```

#### ファイルを削除するコマンド

- rm ファイル名
    - ファイルを削除する。

```shell
[user@localhost ~]$ rm sample1.txt 
[user@localhost ~]$ ls
sampledir     テンプレート  ドキュメント  音楽  公開
ダウンロード  デスクトップ  ビデオ        画像
```

- rm -d ディレクトリ名
    - 空ディレクトリを指定して削除する。

```shell
[user@localhost ~]$ rm -d sampledir
rm: `sampledir` を削除できません: ディレクトリは空ではありません
[user@localhost ~]$ mkdir testdir
[user@localhost ~]$ ls
sampledir  ダウンロード  デスクトップ  ビデオ  画像
testdir    テンプレート  ドキュメント  音楽    公開
[user@localhost ~]$ rm -d testdir
[user@localhost ~]$ ls
sampledir     テンプレート  ドキュメント  音楽  公開
ダウンロード  デスクトップ  ビデオ        画像
```


- rm -d -r ディレクトリ名
    - ディレクトリを指定して、中のファイルを含めて削除する。

```shell
[user@localhost ~]$ rm -d -r sampledir
[user@localhost ~]$ ls
ダウンロード  デスクトップ  ビデオ  画像
テンプレート  ドキュメント  音楽    公開
```

#### ファイルのパーミッション（所有権）を変更するコマンド

- chmod 引数 ファイル名
    - ３桁の引数で１桁目にファイル所有者、２桁目にグループ、３桁目にその他のユーザに、権限を変更する。

```shell
[root@localhost ~]# vi sample.txt
[root@localhost ~]# ls -l
合計 12
-rw-------. 1 root root 1682 11月 16  2022 anaconda-ks.cfg
-rw-r--r--. 1 root root 1710 11月 17  2022 initial-setup-ks.cfg
-rw-r--r--. 1 root root    7 11月 16 16:44 sample.txt
```

|  8進数  |  2進数(rwx)  |  権限  |
| ---- | ---- | ---- |
|  0  |  000  |　権限なし |
|  1  |  001  |  実行  |
|  2  |  010  |  読込　|
|  3  |  011  |  読込+実行  |
|  4  |  100  |  書込　|
|  5  |  101  |  書込+実行  |
|  6  |  110  |  読込+書込  |
|  7  |  111  |  全て  |

```shell
[root@localhost ~]# chmod 666 sample.txt
[root@localhost ~]# ls -l
合計 12
-rw-------. 1 root root 1682 11月 16  2022 anaconda-ks.cfg
-rw-r--r--. 1 root root 1710 11月 17  2022 initial-setup-ks.cfg
-rw-rw-rw-. 1 root root    7 11月 16 16:44 sample.txt
```

#### ファイル及びディレクトリの所有者を変更するコマンド

- chown ユーザ名 ファイル名
    - 指定したファイルの所有者を指定したユーザに変更する

```shell
[root@localhost ~]# chown user sample.txt
[root@localhost ~]# ls -l
合計 12
-rw-------. 1 root root 1682 11月 16  2022 anaconda-ks.cfg
-rw-r--r--. 1 root root 1710 11月 17  2022 initial-setup-ks.cfg
-rw-rw-rw-. 1 user root    7 11月 16 16:44 sample.txt
```

### ファイル内容の表示及び検索するコマンド

#### ファイルの内容を表示するコマンド

- cat ファイル名
    - ファイルの中身を表示する

```shell
[root@localhost ~]# cat sample.txt
sample
```

- less ファイル名
    - スクロール可能な状態で表示する（qで強制終了）

```shell
[root@localhost ~]# less sample.txt 
```

#### ファイルの内容を検索するコマンド

- grep 検索文字列 ファイル名
    - ファイル名から検索文字列が含めれる行を表示する

```shell
[root@localhost ~]# grep a sample.txt
```

