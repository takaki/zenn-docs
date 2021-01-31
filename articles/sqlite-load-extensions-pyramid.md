---
emoji: "💻"
type: "tech"
published: true
title: sqliteのload_extensionsをPyramidで使う
topics: ["pyramid","Python","sqlite","sqlalchemy"]
author: takaki@github
slide: false
---
SQLiteでは数学関数や統計関数などを使うには拡張ライブラリを使う必要があるがPyramidから使うにはちょっと工夫がいった。

# 拡張の導入
以下のURLから extension-functions.c をダウンロードする。
* https://www.sqlite.org/contrib
次に共有ライブラリの作成をする。

```
gcc -fPIC -shared /home/takaki/Desktop/extension-functions.c -o libsqlitefunctions.so
```

sqliteを起動して以下のようにできれば拡張ライブラリはOK。

```
$ sqlite3 
SQLite version 3.13.0 2016-05-18 10:57:30
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
sqlite> select load_extension('./libsqlitefunctions.so');

sqlite> select sin(3.14/2);
0.999999682931835
sqlite> 
```

# Pyramidからの呼出
通常の雛形ファイルから以下のような変更をする。

```python:model/__init__.py
def get_session_factory(engine):
    @event.listens_for(engine, "connect")
    def connect(dbapi_connection, connection_rec):
        dbapi_connection.enable_load_extension(True)
        dbapi_connection.execute("SELECT load_extension('{0}');".format(FULL_PATH_OF_LIB))
    factory = sessionmaker()
    factory.configure(bind=engine)
    return factory
```

* http://abrightersun.de/blog/spatialite-and-sqlalchemy.html
* https://code.ffdn.org/FFDN/ffdn-db/src/master/ffdnispdb/__init__.py

