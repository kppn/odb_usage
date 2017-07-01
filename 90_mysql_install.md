# MYSQL(MariaDB)インストール

## インストール＆起動

```
yum install mysql
yum install mysql-server
systemctl start mariadb
```

## 初期設定

```
mysql_secure_installation
```

`Enter current password for root`はまだrootパスワードを設定していないのですぐEnter。

次に`Set root password?`で`Y`、rootパスワードを設定する。

その他は全てEnter (テスト用のユーザ/テーブルの削除して良いか聞かれているので全てYes)。

## MariaDBのユーザ作成

MariaDBに入る

```
mysql -u root -p
```

現在のユーザ一覧。rootのみ存在する。

```
select user,host,user from mysql.user;
```



## ユーザ作成

testデータベースに対して全ての権限を持ったユーザtestuser作成。

```
grant all on test.* to testuser identified by 'password';
SET PASSWORD FOR 'testuser'=password('test123');
```

# 実験

## データベース作成

```
CREATE DATABASE test;
SHOW DATABASES;
```

## testuserでtestデータベースへ入る
```
mysql -u testuser -p test
```

## テーブル作成
```sql
CREATE TABLE users (
  name TEXT
);

SHOW TABLES;
```

## 挿入
```sql
SELECT * FROM users;

INSERT INTO users values (
  "takashi"
);

SELECT * FROM users;
```

## 更新

```sql
UPDATE users set
  name="John Doe"
    WHERE name="takashi";

SELECT * FROM users;
```

## 削除

```sql
DELETE FROM users
  WHERE name="John Doe";

SELECT * FROM users;
```

## テーブル削除
```sql
DROP TABLE users;

SHOW TABLES;
```
