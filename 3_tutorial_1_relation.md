# relationの例

モデル間にRelationを定義する。  

Programmer has many Tools.


## 後述のMakefileを先に作っておく


## Programmer/Toolモデルのヘッダーファイルを作成

programmer.hxx
```c++
#ifndef __PROGRAMMER_HXX
#define __PROGRAMMER_HXX

#include <string>
#include <vector>
#include <memory>

#include <odb/core.hxx>

class Tool;
using std::shared_ptr;


#pragma db object table("programmers")
class Programmer
{
public:
  // コンストラクタ
  Programmer(const std::string& name) : name_(name) {}

  // nameへのアクセサ
  const std::string& name () const {return name_;}
  void name(std::string& name) {name_ = name;}

  const std::vector<shared_ptr<Tool> > tools() const {return tools_;}
  std::vector<shared_ptr<Tool> >& tools() {return tools_;}

private:
  Programmer() {}

  friend class odb::access;


  // カラム
  #pragma db id auto // idは自動採番
  unsigned long id_;

  std::string name_;

  #pragma db value_not_null unordered
  std::vector<shared_ptr<Tool> > tools_;
};


#pragma db object table("tools")
class Tool
{
public:
  Tool(const std::string& name) : name_(name) {}

  // nameへのアクセサ
  const std::string& name () const {return name_;}
  void name(std::string& name) {name_ = name;}

private:
  Tool() {}

  friend class odb::access;

  #pragma db id auto   // idは自動採番
  unsigned long id_;

  // カラム
  std::string name_;
};


#endif   // __PROGRAMMER_HXX
```

### モデルを使うmainコードを作成

driver.cxx

```cpp
#include <memory>   // std::auto_ptr
#include <iostream>

#include <odb/database.hxx>
#include <odb/session.hxx>
#include <odb/transaction.hxx>

#include <odb/mysql/database.hxx>

#include "programmer.hxx"
#include "programmer-odb.hxx"

using namespace std;
using namespace odb::core;


int main(int argc, char ** argv)
{
  auto_ptr<database> db (new odb::mysql::database (argc, argv));

  {
    Programmer takashi("Takashi");

    shared_ptr<Tool> takashi_tool1(new Tool("GCC Compiler"));
    takashi.tools().push_back(takashi_tool1);

    shared_ptr<Tool> takashi_tool2(new Tool("Object-relational mapping"));
    takashi.tools().push_back(takashi_tool2);


    Programmer god("God");
    shared_ptr<Tool> god_tool1(new Tool("Earth"));
    god.tools().push_back(god_tool1);


    transaction t(db->begin ());

    db->persist(*takashi_tool1);
    db->persist(*takashi_tool2);
    db->persist(takashi);

    db->persist(*god_tool1);
    db->persist(god);

    t.commit ();
  }

  {
    transaction t (db->begin());


    typedef odb::query<Programmer> query;
    typedef odb::result<Programmer> result;
    result r( db->query<Programmer>());

    for (result::iterator prog = r.begin(); prog != r.end (); ++prog) {
      std::cout << prog->name() << std::endl;

      for (auto tool : prog->tools()) {
        std::cout << "  " << tool->name() << std::endl;
      }
    }

    t.commit();
  }

}
```

読み出しのコードで、Programmerを読みだしただけで、各ProgrammerのToolが読み出されて
いることに注意(Eagar loading)。


## ODBでコード生成・コンパイル
```
make
```

## テーブル作成
```
make migrate
```

programmers, programmers_tools, toolsのテーブルが出来てる。  
programmers_toolsはRelationのテーブル。


## 実行
```
make exec
```

## DBの中を確認
```
make query
```

```
echo -n 'select * from programmers;'       | mysql --user=testuser --database=test -ptest123
id	name
13	Takashi
14	God

echo -n 'select * from tools;'             | mysql --user=testuser --database=test -ptest123
id	name
19	GCC Compiler
20	Object-relational mapping
21	Earth

echo -n 'select * from programmers_tools;' | mysql --user=testuser --database=test -ptest123
object_id	value
13	19
13	20
14	21
```

programmers_toolsを介して、programmersとtoolsテーブルのレコード間でRelationが
組まれている。



## makefile

```makefile
all:
	odb --std c++11 -d mysql --generate-query --generate-schema --generate-session --default-pointer std::shared_ptr programmer.hxx
	g++ -std=c++11 -c programmer-odb.cxx
	g++ -std=c++11 -c driver.cxx
	g++ -std=c++11 -o driver driver.o programmer-odb.o -lodb-mysql -lodb

migrate:
	mysql -p --user=testuser --database=test < programmer.sql

exec:
	./driver --user testuser --database test --password test123

query:
	echo -n 'select * from programmers;'       | mysql --user=testuser --database=test -ptest123
	echo -n 'select * from tools;'             | mysql --user=testuser --database=test -ptest123
	echo -n 'select * from programmers_tools;' | mysql --user=testuser --database=test -ptest123

delete:
	echo -n 'delete from programmers;'       | mysql --user=testuser --database=test -ptest123
	echo -n 'delete from tools;'             | mysql --user=testuser --database=test -ptest123
	echo -n 'delete from programmers_tools;' | mysql --user=testuser --database=test -ptest123
```
