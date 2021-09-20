---
title: "MySQL CLIの基本操作"
emoji: "🐬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["DB", "MySQL"]
published: true
---

# ローカルで MySQL を起動する

MySQL サーバーの起動/停止は `mysql.server start/stop` コマンドを利用する。

# cli での基本操作

```sh
# サーバーへの接続
$ mysql -u user_name -h 127.0.0.1 --port 3306 -p
Enter password: [パスワードを入力]

# データベースの一覧
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

# ユーザーの一覧表示
mysql> select user, host from mysql.user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
4 rows in set (0.00 sec)

# 操作するデータベースを指定
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

# ユーザーの作成
mysql> create user 'admin'@'localhost' identified by 'password';
Query OK, 0 rows affected (0.02 sec)

mysql> select user, host from user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| admin            | localhost |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
5 rows in set (0.00 sec)
```

# user 権限の付与

データベース内のフルアクセス権限を付与する場合は以下のコマンド。

```sh
# ユーザーに権限を付与する
mysql> GRANT ALL PRIVILEGES ON * . * TO 'admin'@'localhost';

# 新しいユーザーに設定する権限を確定したら、すべての権限のリロードが必要。
mysql> FLUSH PRIVILEGES;
```

一般的なアクセス権限の一覧。

| 権限           | 説明                                                                                                                                                                         |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ALL PRIVILEGES | 前述したように、MySQL ユーザーは指定されたデータベースへフルアクセスができます（または、データベースが選択されていない場合は、システム全体のグローバルアクセスができます）。 |
| CREATE         | 新しいテーブルまたはデータベースを作成できます。                                                                                                                             |
| DROP           | テーブルまたはデータベースを削除できます。                                                                                                                                   |
| DELETE         | テーブルから行を削除できます。                                                                                                                                               |
| INSERT         | テーブルに行を挿入できます。                                                                                                                                                 |
| SELECT         | SELECT コマンドを使用してデータベースを読み取ることができます。                                                                                                              |
| UPDATE         | テーブルの行を更新できます。                                                                                                                                                 |
| GRANT OPTION   | 他のユーザーの権限の付与または削除ができます。                                                                                                                               |

特定ユーザーに対して、権限を付与する場合は以下のコマンドで可能。(権限取り消しの場合は、GRANT を REVOKE に変えると可能)

```sh
mysql> GRANT type_of_permission ON database_name.table_name TO 'username'@'localhost';
```

ユーザーの権限は以下のコマンドで確認できる。

```sh
mysql> SHOW GRANTS FOR 'username'@'localhost';
```

# 参考リンク

[MySQL で新しいユーザーを作成して権限を付与する方法](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql-ja)
