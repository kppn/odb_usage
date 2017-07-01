# personテーブルの例

personテーブルにレコードを追加し、読みだす例。  
テーブルは以下
```sql
person
  id
  name
  age
```

## Personモデルのヘッダーファイルを作成

カラムのアクセサは、ファイルを分けるほどではないためここで実装する。

person.hxx
```c++ : person.hxx
#ifndef __PERSON_HXX
#define __PERSON_HXX

#include <string>

#include <odb/core.hxx>

#pragma db object table("people")
class Person
{
public:
  // コンストラクタ
  Person (const std::string& name, unsigned short age) : name_(name), age_(age) {}

  // nameへのアクセサ
  const std::string& name () const {return name_;}
  void name(std::string& name) {name_ = name;}

  // ageへのアクセサ
  unsigned short age () const {return age_;}
  void age (unsigned short age) {age_ = age;}

private:
  #pragma db table("person")

  Person() {}  // deny default contructor

  friend class odb::access;

  #pragma db id auto   // idは自動採番
  unsigned long id_;

  // カラム
  std::string name_;
  unsigned short age_;
};

#endif   // __PERSON_HXX
```


## ODBでコード生成
```
odb -d mysql --generate-query --generate-schema person.hxx
```

以下のファイルが出来る
* person-odb.cxx
* person-odb.hxx
* person-odb.ixx
* person.sql

person.sqlは、テーブル作成のsqlファイル。  
これをDBに読み込ませてテーブル作成。
```
mysql -p --user=testuser --database=test < person.sql
```

mysqlでテーブルが出来たことを確認。


## モデルを使うmainコードを作成

driver.cxx
```cpp : driver.cxx
#include <memory>   // std::auto_ptr
#include <iostream>

#include <odb/database.hxx>
#include <odb/transaction.hxx>
#include <odb/mysql/database.hxx>
#include "person.hxx"
#include "person-odb.hxx"

using namespace std;
using namespace odb::core;


int main(int argc, char ** argv)
{
  auto_ptr<database> db (new odb::mysql::database (argc, argv));

  // モデルを作成
  Person john("John", 33);
  Person jane("Jane", 32);

  transaction t(db->begin ());
  // DBへ保存
  unsigned long john_id = db->persist(john);
  unsigned long jane_id = db->persist(jane);
  t.commit ();

  // DBから読み出す
  typedef odb::query<Person> query;
  typedef odb::result<Person> result;
  {
    transaction t (db->begin ());

    // 30歳以上の人だけ
    result r(db->query<Person> (query::age > 30));
    for (result::iterator i = r.begin(); i != r.end (); ++i) {
      cout << "Hello, " << i->name () << "!" << endl;
    }

    t.commit();
  }
}

```

DBを覗いてみよう。  
JohnとJaneがpeopleテーブルに入っている。


## makefile

```makefile
all:
	odb -d mysql --generate-query --generate-schema person.hxx
	g++ -std=c++11 -c person-odb.cxx
	g++ -std=c++11 -c driver.cxx
	g++ -std=c++11 -o driver driver.o person-odb.o person_impl.o -lodb-mysql -lodb

migrate:
	mysql -p --user=testuser --database=test < person.sql

exec:
	./driver --user testuser --database test --password test123
```
