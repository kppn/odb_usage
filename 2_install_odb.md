# インストール for Centos 7.3

## 必要ライブラリインストール
```
yum install gcc-plugin-devel
yum install mysql-devel
```

## ダウンロード

```
wget http://www.codesynthesis.com/download/odb/2.4/libodb-2.4.0.tar.bz2
tar jxvf libodb-2.4.0.tar.bz2

wget http://www.codesynthesis.com/download/odb/2.4/libodb-mysql-2.4.0.tar.bz2
tar jxvf libodb-mysql-2.4.0.tar.bz2

wget http://www.codesynthesis.com/download/libcutl/1.10/libcutl-1.10.0.tar.bz2
tar jxvf libcutl-1.10.0.tar.bz2

wget http://www.codesynthesis.com/download/odb/2.4/odb-2.4.0.tar.bz2
tar jxvf odb-2.4.0.tar.bz2
```

## インストール

## libcutl

```
cd libcutl-1.10.0
./configure
make
make install
ls /usr/local/lib/libcutl*
```

## libodb

```
cd libodb-2.4.0
./configure
make
make install
ls /usr/local/lib/libodb*
```

## libodb-mysql

```
./configure
make
make install
ls /usr/local/lib/libodb-mysql*
```

## odb

```
./configure
make
make install
ls /usr/local/libexec/odb
ls /usr/local/bin/odb
```

## ライブラリ依存更新

```
echo '/usr/local/lib' > /etc/ld.so.conf.d/usr-local-lib.conf
ldconfig
```

## odbコマンド起動確認

```
odb
```
odb: error: no database specified with the --database option
と出たら成功
