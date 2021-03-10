# Giztech 環境構築手順書
Vagrantを使用したLaravel環境を構築するための手順書です。

## 仕様
1. **Vagrant**を使用すること
2. ipは**192.168.33.19**とすること
3. OSは**CentOS**
4. DBは**MySQL5.7**
5. Webサーバーは**Nginx**
6. PHPのバージョンは**7.3**
7. Laravelのバージョンは**6.0**

### バージョン情報
|環境|使用技術|バージョン|
|:--:|:--:|:--:|
|AppServer|PHP|7.3|
|WebServer|Nginx||
|DataBase|MySQL|5.7|
|FrameWork|Laravel|6.0|
|OS|CentOS|7系|

**その他使用技術**
- VirtualBox

## 作業準備

### 1. 環境構築 (仮想マシン立ち上げまで)
以下のコマンドを実行しVagrantディレクトリを作成します。
```
$ mkdir vagrant_dir && cd vagrant_dir
```

CentOSのboxを使用するために以下のコマンドを実行します。(仮想マシンの初期化)
```
$ vagrant init centos/7
```
成功すれば以下のように表示されます。
```
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.

# 日本語訳
`Vagrantfile`がこのディレクトリに配置されました。
これで、最初の仮想環境を「vagrant up」する準備が整いました。 
Vagrantの使用の詳細については、Vagrantfileのコメントと
`vagrantup.com`のドキュメントをお読みください。
```

カレントディレクトリにはVagrantfileができています。
```
$ ls
Vagrantfile
```
中身を変更します。
```vagrantfile:Vagrantfile
# ① 26行目 コメントイン
  config.vm.network "forwarded_port", guest: 80, host: 8080

# ② 35行目 コメントイン
  config.vm.network "private_network", ip: "192.168.33.19"

# ③ 46行目　コメントイン&編集
  # config.vm.synced_folder "../data", "/vagrant_data"
  ↓
  config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
```
--- 
**以下解説(コメントアウトのGoogle翻訳)**
- **① ポートの割り当て**
```
config.vm.network "forwarded_port", guest: 80, host: 8080
```
>ホストマシンのポートからマシン内の特定のポートにアクセスできるようにする転送ポートマッピングを作成します。上の例では、「localhost:8080」にアクセスすると、ゲストマシンのポート80にアクセスします。
注:これにより、開いているポートへのパブリックアクセスが有効になります。
仮想マシンのWebサーバーのポート80が、ホストマシンのポート8080に割り当てられるように設定しています。

- **② 仮想ホストのIPアドレスの設定**
```
config.vm.network "private_network", ip: "192.168.33.19"
```
>特定のIPを使用してマシンへのホストのみのアクセスを許可するプライベートネットワークを作成します。

- **③ 共有フォルダのパスを指定**
```
config.vm.synced_folder "./", "/vagrant", type:"virtualbox
```
>追加のフォルダーをゲストVMと共有します。最初の引数は、ホスト上の実際のフォルダーへのパスです。 2番目の引数は、フォルダーをマウントするゲストのパスです。また、オプションの3番目の引数は、不要なオプションのセットです。

参考URL: [Vagrantで開発環境を用意したときにVagrantfileに入れた設定メモ](https://qiita.com/watyanabe164/items/cdb3949a1a4f6c56bb31)

---
#### Vagrantプラグインのインストール
**vagrant-vbguest**
vagrant-vbguestは初めに追加したBoxの中にインストールされているGuest Additionsというもののバージョンを、VirtualBoxのバージョンに合わせて最新化してくれるプラグインです。 
以下のコマンドを実行してインストールします。
```
$ vagrant plugin install vagrant-vbguest
```

以下のコマンドを実行するとインストールされたか確認できます。
```
$vagrant plugin list
vagrant-vbguest (0.29.0, global)
```

**Sahara**
仮想環境のバージョン管理ができるプラグインです。
必要に応じて入れてください。
```
$ vagrant plugin install sahara
```
使用する時
```
$ vagrant sandbox on
```
保存する時
```
$ vagrant sandbox commit
```
保存地点まで戻す時
```
$ vagrant sandbox rollback
```
止める時
```
$ vagrant sandbox off
```
参考URL: [VagrantにSaharaを導入](https://qiita.com/sudachi808/items/09cbd3dd1f5c25c23eaf)

---
#### Vagrantを使用してゲストOSの起動
```
$ vagrant up
```
※ 立ち上がるまで時間がかかります。

#### =='vagrant up'でエラーが出た！！==
エラー分の切り抜きは以下の通りです。
```
(省略)
No package kernel-devel-3.10.0-1127.el7.x86_64 available.
Error: Nothing to do
Unmounting Virtualbox Guest Additions ISO from: /mnt
umount: /mnt: not mounted
（省略)
```

~~vbguest使ってみたけどダメでした。~~

`No package kernel-devel-3.10.0-1127.el7.x86_64 available.` 
ここで止まっているので以下の記事を参考にした。 
[# Vagrant + VirtualBOx で 最新のCentOS7 vbox(centos/7 2004.01)でマウントできない問題](https://qiita.com/mao172/items/f1af5bedd0e9536169ae)

下記のコマンドを実行してみます。
```
$ vagrant ssh (ゲストOSにログイン)
$ sudo yum -y update kernel
$ exit (ゲストOSからログアウト)
$ vagrant reload --provision
```
うまくいったようです。
***

### 2. ゲストOSのログイン
以下のコマンドを実行してゲストOSへログインできます。
```
$ vagrant ssh
```
ホストOSとゲストOSどっちでコマンドを実行するのか間違えないように!! 

SSHの設定確認をします。
※ ゲストOSでログインしている場合は`$ exit`で一旦ログアウトしてくだい。
```
(ホストOS)
$ vagrant ssh-config
```
***

## 仮想環境構築
**以下ゲストOS上での操作になります。** 
```
$ vagrant ssh
```
### 1. パッケージをいれる
パッケージのインストールは以下のコマンドで実行できます。
```
$ sudo yum -y install パッケージ名
$ sudo yum -y groupinstall "導入する名称"
```
#### 開発環境ツールのインストール
開発環境に必要なパッケージをまとめて入れます。
```
$ sudo yum -y groupinstall "development tools"
```
***

#### PHPのインストール
今回は**PHP7.3**を使用します。
```
$ sudo yum -y install epel-release wget
$ sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
$ sudo rpm -Uvh remi-release-7.rpm
$ sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip
```
インストール完了確認
```
$ php -v
```
***

#### Composerのインストール
```
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ php composer-setup.php
$ php -r "unlink('composer-setup.php');"

# どのディレクトリにいてもcomposerコマンドを使用できるようfileの移動を行います
$ sudo mv composer.phar /usr/local/bin/composer
```
インストール完了の確認
```
$ composer -v
```
***

### 2. Laravelプロジェクトの作成
今回は**Laravel6.0**を使用します。
```
$cd /vagrant
$ composer create-project --prefer-dist laravel/laravel=6.0 sample_app
```
sample_appがプロジェクト名になります。
適宜任意のプロジェクト名に変更してください。
※ 今回は"**sample_app**"というプロジェクト名で進めます。

インストールできたか確認をします。
```
$ cd sample_app
$ php artisan --version
```

authの実装はLaravel6.xからlaravel/uiという別パッケージを使います。
```
$ composer require laravel/ui:^1.0 --dev
$ php artisan ui bootstrap --auth
```
確認するために立ち上げてみます。
```
$ sudo systemctl start php-fpm
```
[http://192.168.33.19:8000](http://192.168.33.19:8000)にアクセスしてLaravelのホームが表示されたらOKです。

**参考URL**
- [更新！！Laravel6/7 (laravel/ui)でのLogin機能の実装方法〜MyMemo](https://qiita.com/daisu_yamazaki/items/a914a16ca1640334d7a5)
- [laravel.com](https://laravel.com/docs/6.x/frontend#introduction)
- [Vagrant + VirtualBox + Centos7でLaravel6環境構築とインストール・起動まで](https://webrogram.com/entries/vagrant-centos7-environment)

***

### 3. Nginx(Webサーバー)のセットアップ
#### Nginxのインストール
コマンドを実行して以下のファイルを作成します。
```
$ sudo vi /etc/yum.repos.d/nginx.repo
```
内容は以下の内容を記述してください。
```repo=
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1
```
保存した後、以下のコマンドを実行してNginxをインストールします。
```
$ sudo yum install -y nginx
$ nginx -v
```
バージョンの確認ができたらNginxを一度起動させます。
```
$ sudo systemctl start nginx
```
ブラウザで[http://192.168.33.19](http://192.168.33.19)を開き、NginxのWelcomeページが表示されることを確認する。
***

#### Laravelでの設定
viエディタでnginxの設定ファイルを編集します。
```
$ sudo vi /etc/nginx/conf.d/default.conf
```
```conf=
server {
  listen       80;
  server_name  192.168.33.19; # 指定のIPアドレス
  root /vagrant/sample_app/public; # 追記
  index  index.html index.htm index.php; # 追記

  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;

  location / {
      #root   /usr/share/nginx/html; # コメントアウト
      #index  index.html index.htm;  # コメントアウト
      try_files $uri $uri/ /index.php$is_args$args;  # 追記
  }

  # 省略

  # 該当箇所のコメントを解除し、必要な箇所には変更を加える
  # 下記は root を除いたlocation { } までのコメントが解除されていることを確認してください。

  location ~ \.php$ {
  #    root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
      include        fastcgi_params;
  }

  # 省略
```

次に **php-fpm** の設定ファイルを編集します。
```
$ sudo vi /etc/php-fpm.d/www.conf
```
```conf=
;24行目近辺
user = apache
# ↓ 以下に編集
user = vagrant

group = apache
# ↓ 以下に編集
group = vagrant
```
以下のコマンドを実行して **vagrant** というユーザーでもログファイルへの書き込みができる権限を付与します。
```
$ cd /vagrant/sample_app
$ sudo chmod -R 777 storage
```

最後に**nginx**と**php-fpm**の再起動を行います。
```
$ sudo systemctl restart nginx
$ sudo systemctl restart php-fpm
```
***

**"Forbidden 403" が出た時** 
SELinuxによって制限されていることがあります。
>SELinux (Security-Enhanced Linux) とは、システムにアクセス可能なユーザーをより詳細に制御できるようにする、Linux® システム 用のセキュリティ・アーキテクチャです。
>引用サイト: [SELinux とは](https://www.redhat.com/ja/topics/linux/what-is-selinux)

以下のコマンドを実行して設定を編集します。
```
$ sudo vi /etc/selinux/config
```
以下のように編集してください。
```=
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.

SELINUX=disabled # <= enforcingからdisabledに編集
```
設定反映のために再起動
```
(ゲストOSをログアウト)
$ exit

(ホストOS)
$ vagrant reload
$ vagrant ssh

(ゲストOS)
$ sudo systemctl start nginx
$ sudo systemctl start php-fpm
```
***

### 4. MySQL(データベース)のセットアップ
今回は**MySQL5.7**を使用します。
#### MySQ5.7Lのインストール
```
$ sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
$ sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm
$ sudo yum install -y mysql-community-server
```
インストールできたか確認します。
```
$ mysql --version
```
***

#### MySQLの接続
```
$ sudo systemctl start mysqld
```

**DBパスワードを緩くする場合(開発用)**
```
$ sudo vi /etc/my.cnf
```
```=
# 省略

[mysqld]

# 省略

# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# 下記の一行を追加
validate-password=OFF
```
```
$ sudo systemctl restart mysqld
```

注意: ステージング環境や本番環境で使用するDBにはポリシーを遵守したパスワードを設定してください。


MySQLの初期パスワードの確認をします。(***のところ)
```
$ sudo cat /var/log/mysqld.log | grep 'temporary password'
2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: *******
```
MySQLにログインしてパスワードの再設定を行います。
```
$ mysql -u root -p
Enter password:
mysql > set password = "新たなパスワードを入力";
```
***
#### テーブルの作成
```
mysql > CREATE DATABASE sample_app;
```
Query OK と表示されればOKです。
`exit`してDBからログアウトします。
***

#### Laravelの環境ファイル編集
sample_appディレクトリに移動して`.env`ファイルを以下のように編集します。
```=
# 変更
DB_DATABASE=laravel
	↓
DB_DATABASE=sample_app	#(MySQLで作成したデータベース)

# 追記
DB_PASSWORD=
	↓
DB_PASSWORD=新しく登録したパスワード
```

**参考URL**
- [vagrant環境化のLaravelでインストール直後にThe stream or file “/vagrant/xxxxx/storage/logs/laravel.log” could not be opened: failed to open stream: Permission denied](https://hiroslog.com/post/96)
- [【vagrant】共有フォルダのパーミッションで悩んだ話【chmodできない】](http://ism1000ch.hatenablog.com/entry/2014/04/05/232935)
***

#### migrate実行
以下のコマンドを実行してデータベースにテーブルを作成します。
```
$ cd /vagrant/sample_app
$ php artisan migrate
```
***

### うまくいかない時は
うまく動作しない時は一度立ち上げ直すのも一つの手です。
設定を変えたものの再起動によって初めて反映される場合があるためです。
- 再起動
```
(ゲストOSにいる時)
$ sudo systemctl reload [service]

(ホストOSにいる時: 仮想環境の再起動)
$ vagrant reload
```
- 立ち上げ(ゲストOS)
```
(ホストOS)
$ vagrant ssh

(ゲストOS)
$ sudo systemctl start nginx
$ sudo systemctl start php-fpm
$ sudo systemctl start mysqld
```
**その他使いそうな仮想マシン操作コマンド**
- 仮想マシンの一時停止
```
$ vagrant suspend
```
- 仮想マシンの一時停止からの復帰
```
$ vagrant resume
```

- 仮想マシンを終了させる時
```
$ vagrant halt
```
***
それでもわからない場合はGoogleで調べましょう。
エラー文コピペでも十分に効果があります。
***

## その他参考URL
- [GizTech](https://giztech.gizumo-inc.work/)
- [【まとめ】Vagrant コマンド一覧](https://qiita.com/oreo3@github/items/4054a4120ccc249676d9)