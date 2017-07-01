# なぜODBか

Relational DBへアクセスするアプリケーションでは、一般に次の操作が必要になる。
テーブルのCRUD操作
テーブル間のRelation
オブジェクト指向言語でこれを表現する場合、各テーブルをクラスとしてラップする。

例えば、DBにusersテーブルが有った場合を想定しよう。

```sql
CREATE TABLE users(
  id int NOT NULL AUTO_INCREMENT,
  name TEXT NOT NULL,
  age INT
)
```

このテーブルへの操作は次のように行いたい。

```c++
// 新しいユーザを作成しDBへ保存
User user("takashi", 10);  
user.save();               // DBへ保存

// ...

// nameが"takashi"のレコードを検索しオブジェクト作成、更新して保存
user = User::find("name", "takashi")
user.age(20);
user.save();
```

過去に多くのアプリケーションがこれを実現するクラスを自前で実装してきたが、自動化が模索された結果、Object Relational Mappingというパターンが生まれた。
