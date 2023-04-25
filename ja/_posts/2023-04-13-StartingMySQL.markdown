---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: content_2
title: MySQLをうす〜く使ってみる

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
#author: "peartrees"
# multiple category is not supported
category: programming
# multiple tag entries are possible
tags: [programming,database,SQL,MySQL]
# thumbnail image for post
img: ":post_pic3.jpg"
# disable comments on this page
#comments_disable: true

# publish date
date: 2023-04-13 17:45:00 +0900

# seo
# if not specified, date will be used.
#meta_modify_date: date: 2023-04-13 17:45:00 +0900
# check the meta_common_description in _data/owner/[language].yml
#meta_description: ""

# optional
# please use the "image_viewer_on" below to enable image viewer for individual pages or posts (_posts/ or [language]/_posts folders).
# image viewer can be enabled or disabled for all posts using the "image_viewer_posts: true" setting in _data/conf/main.yml.
#image_viewer_on: true
# please use the "image_lazy_loader_on" below to enable image lazy loader for individual pages or posts (_posts/ or [language]/_posts folders).
# image lazy loader can be enabled or disabled for all posts using the "image_lazy_loader_posts: true" setting in _data/conf/main.yml.
#image_lazy_loader_on: true
# exclude from on site search
#on_site_search_exclude: true
# exclude from search engines
#search_engine_exclude: true
# to disable this page, simply set published: false or delete this file
#published: false
---

この記事は，Qiitaに以前投稿した記事：[MySQLをうす〜く使ってみる](https://qiita.com/peartrees/items/e3ae29fed26f0365b130)を移植したものになります．

---

## 概要
データベースを使ってみたいという安直な理由で**MySQL**を触ってみます．

本当に1ミリも知識がない（←ここ重要）&環境もないため，installからデータベース操作まで，もがきながらやってみます．
（とにかく使ってみたいというだけなので，細かいところは全無視でよく分かっていなくても進めています）

本記事はその備忘録として作成します．


## 1. MySQLのInstallと初期設定

```terminal:MySQLのinstall
brew install mysql
```

遅くなりましたが，筆者はmacユーザーなのでmac OSでの環境構築になります．

```terminal:MySQLのversion確認
mysql --version
-> mysql  Ver 8.0.27 for macos11.6 on x86_64 (Homebrew)
```

次のコマンドを実行すると，MySQLを起動することができます．

```terminal:MySQLの起動（いずれか1つでOK）
brew services start mysql
mysql.server start
```
止めたい時は，次のコマンドを実行しましょう!

```terminal:MySQLの停止
brew services stop mysql
```

初回は，```mysql_secure_installation```でパスワードを設定する必要があります．割と強固なものにした方が良さそうです（ポリシー的に）
セキュリティについて色々聞かれますが，基本「yes」で大丈夫だと思います．

```terminal:MySQLの初期設定（初回install時のみ）
mysql_secure_installation
```

次のコマンドを実行してログインしてみましょう!

```terminal:MySQLへのログイン
mysql --user=root --password
```

ログインが成功したら，MySQLを使う準備はバッチリです！
次の章から実際に中身を覗いてみましょう！

## 2. MySQLを使ってみる

MySQLの主なコマンドから確認していきます．
最初は```show databases```です．
データベースを一覧表示するためのコマンドです．

何か色々表示されていますが，よく分からないので今回はパスです．

```terminal:データベースの一覧表示
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.04 sec)
```

続いては```CREATE DATABASE```です．
新しくデータベースを作成したい時に使います．
実行してみると，新しく追加されていることが分かります！

```terminal:データベースの新規作成
mysql> CREATE DATABASE db_sample;
Query OK, 1 row affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| db_sample          |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

一覧表示した中からデータベースを選択したい場合には```use db_sample```のように実行します．
また，データベース中のテーブルを表示したい場合には```show tables```を実行しましょう．
現在は作成してから何もいじっていないので**Empty set**になっていますね．

```terminal:データベースの変更とテーブル表示
mysql> use db_sample;
Database changed
mysql> show tables;
Empty set (0.00 sec)
```

続いてはお馴染み```CREATE TABLE```文です．
あるデータベース中で新しくテーブルを作成したいときに実行します．
今回はidとnameという2つのカラムを用意します．idはkeyであり，auto_incrementを引数に加えることで，データを追加する度に+1される設定にしています（初期値は１）．nameは文字列ならオッケーとしています．

```terminal:CREATETABLE文
mysql> CREATE TABLE users (id INT AUTO_INCREMENT, name TEXT, PRIMARY KEY (id));
Query OK, 0 rows affected (0.03 sec)
```

作成したテーブルについて詳細に知りたい時には```DESCRIBE```文を使います．
keyが何か，default値は何か，typeの指定は何かを一眼で把握することができます．

```terminal:DESCRBE文
mysql> DESCRIBE users;
+-------+------+------+-----+---------+----------------+
| Field | Type | Null | Key | Default | Extra          |
+-------+------+------+-----+---------+----------------+
| id    | int  | NO   | PRI | NULL    | auto_increment |
| name  | text | YES  |     | NULL    |                |
+-------+------+------+-----+---------+----------------+
2 rows in set (0.02 sec)
```

続いては```INSERT INTO```文です．新しくデータを追加したい時に使います．
今回はnameにtestという値を持ったuserを追加してみます．
おまけに結果の抽出も行ってみましょう.テーブルからデータを取り出したい時には```SELECT```文を実行します．
SELECT文は奥が深いので詳細については省きます...

```terminal:INSERT文とSELECT文
mysql> INSERT INTO users(name) VALUES ('test');
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM users;
+----+------+
| id | name |
+----+------+
|  1 | test |
+----+------+
1 row in set (0.00 sec)
```

## 3. 権限関係
本章では，MySQLのセキュリティポリシーや権限関係について見ていこうと思います．

MySQLでは，非常に厳しいセキュリティポリシーが設定されており，パスワードの要求水準が極めて高いです（長いパスワードを設定して入れなくなりがち）．
趣味で使う分にはそこまで高いセキュリティ基準にしなくても良くない？ということで今回は設定を変えて行こうと思います．
まずは，初期の設定がどんなものか```SHOW variables like 'validate_password%'```を実行してみましょう！

```terminal:passwordのvalidation
mysql> SHOW variables like 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | X      |
| validate_password.dictionary_file    |        |
| validate_password.length             | X      |
| validate_password.mixed_case_count   | X      |
| validate_password.number_count       | X      |
| validate_password.policy             | X      |
| validate_password.special_char_count | X      |
+--------------------------------------+--------+
7 rows in set (0.02 sec)
```

上記でかなりタイトな設定になっていることが分かったので，自分なりの設定に変えてみましょう．変更したい時は```set global XXX```を実行します．
変更後もう一度```SHOW variables```を実行すると，修正されていることが分かります．

```terminal:global変数の変更
mysql> set global validate_password.length={hoge};
mysql> set global validate_password.policy={hoge};
Query OK, 0 rows affected (0.00 sec)
```

続いては権限関係を見ていきます．
```CREATE USER```文を実行すると，新規のユーザーを作成することができます．

```terminal:CREATEUSER文
mysql> CREATE USER 'hoge@localhost' IDENTIFIED BY 'password';
Query OK, 0 rows affected (0.01 sec)
```

続いて```GRANT```文を見ていきます．このコマンドは権限を委譲する際に使われるもので，ユーザーごとにアクセスできるデータベース，機能を制限することができます．非常に細かく設定することができ，意図しないユーザーが意図しない操作を行えない仕組みがなされています．
今回はあるユーザーにあるデータベースに関する全権限を付与してみます．

```terminal:GRANT文
mysql> grant all on sample_db.* to hoge@localhost;
Query OK, 0 rows affected (0.01 sec)
```

あるユーザーに与えられた権限の一覧を見たい時には```SHOW GRANTS```を実行します．
先ほど全権限が付与されたので，```ALL PRIVILEGES```となっていることが分かります．

```terminal:あるユーザーへの権限一覧
mysql> show grants for hoge@localhost;
+--------------------------------------------------------------------+
| Grants for hoge@localhost                                          |
+--------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON `sample_db`.* TO `hoge`@`localhost`        |
+--------------------------------------------------------------------+
2 rows in set (0.00 sec)
```

今回はここまで．

本当に浅く触っただけでしたが，データベースの仕組みを完璧に理解できました（できていない）
次回以降は実際にwebアプリに組み込んだりと，何かしら形にできたらいいなあと考えています．


# 参考文献等
本記事を作成するに当たって，以下のサイトを参考にしました．
1. [MySQL公式ドキュメント](https://dev.mysql.com/doc/refman/8.0/en/)
