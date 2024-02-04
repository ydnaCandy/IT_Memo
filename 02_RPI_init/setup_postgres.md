# Raspberry Pi　OSにPostgreSQLをインストールする

## 参考
- https://raspi.taneyats.com/entry/install-postgresql

## 環境
```bash
$ uname -a
>>> Linux raspberrypi 6.1.21-v8+ #1642 SMP PREEMPT Mon Apr  3 17:24:16 BST 2023 aarch64 GNU/Linux

$ lsb_release -a
>>> No LSB modules are available.
>>> Distributor ID: Debian
>>> Description:    Debian GNU/Linux 11 (bullseye)
>>> Release:        11
>>> Codename:       bullseye
```


## セットアップ手順

### postgreSQLのインストール
```bash
# postgresのインストール
$ sudo apt install postgresql

# パスワードを設定する
$ sudo passwd postgres

# postgres用のユーザーを追加
$ su - postgres
>>> postgres@raspberrypi:~$ 

# ユーザーの作成（Linuxのユーザー名と合わせる）
pg$ createuser [username]

# データベースを作成する
pg$ createdb sample

# DBに接続する
pg$ su - [username] # 通常ユーザーに戻る
>>> $

# DBにアクセスする
$ psql sample
sample=> 
sample=> quit;
$ 

```

### 対向認証を解除する
- 初期設定ではpostgreSQLのユーザー名とLinux上でのユーザー名が一致していないとデータベースに接続できない
```bash
# ユーザー名が一致していないとエラーになる
$ psql -U postgres
psql: error: FATAL:  Peer authentication failed for user "postgres"

# 設定ファイルを開く
$ sudo nano /etc/postgresql/13/main/pg_hba.conf 
```

```conf
# "local" is for Unix domain socket connections only
# ↓コメントアウトして
# local   all             all                                     peer

# 同じ内容でpeerをmd5にする
local   all             all                                     md5
```

```bash
# postgresを再起動
$ sudo service postgresql restart

# postgresユーザでユーザにパスワードを付与
$ su - postgres
pg$ psql
postgres=# ALTER ROLE [ユーザー名] PASSWORD ’[パスワード]’;
postgres=# \q

pg$ su - [username] # 通常ユーザーに戻る
>>> $

# データベースに接続してみる
$ psql -U [ユーザー名] [DB名]
sample=>
sample=> \q
```

### 外部接続を許可する
- 同セグメント内での接続許可する
```bash
$ sudo nano /etc/postgresql/13/main/postgresql.conf
```

```conf
# コメントアウトを外して有効にする
# - Connection Settings -
listen_addresses = '*'
```

```bash
$ sudo nano /etc/postgresql/13/main/pg_hba.conf
```

```conf
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
host    all             all             [接続許可するIP]          md5 # 追加
```

何かしらのクライアントソフトで接続してみて、データベースが開けば成功

## アンインストール手順

```bash
# パッケージリストの確認
$ dpkg -l | grep postgres
>>> ii  postgresql                           13+225                           all          object-relational SQL database (supported version)
>>> ii  postgresql-13                        13.13-0+deb11u1                  arm64        The Worlds Most Advanced Open Source Relational Database
>>> ii  postgresql-client-13                 13.13-0+deb11u1                  arm64        front-end programs for PostgreSQL 13
>>> ii  postgresql-client-common             225                              all          manager for multiple PostgreSQL client versions
>>> ii  postgresql-common                    225                              all          PostgreSQL database-cluster manager


# アンインストール
sudo apt remove --purge postgresql -y
sudo apt remove --purge postgresql-13 -y
sudo apt remove --purge postgresql-13-client -y
sudo apt remove --purge postgresql-client-common -y
sudo apt remove --purge postgresql-common -y

# 再起動
sudo reboot now

# postgresユーザーの削除
sudo userdel postgres

# ディレクトリの削除
sudo rm -r /var/lib/postgresql

```