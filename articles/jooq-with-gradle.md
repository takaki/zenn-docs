---
emoji: "💻"
type: "tech"
published: true
title: jOOQをGradleで設定する
topics: ["Java","gradle","jOOQ","ORM"]
author: takaki@github
slide: false
---
JavaのORMであるjOOQをGradleで設定する方法の説明。参考に上げてあるQiitaの記事もあるし元のマニュアルも丁寧ではあるがMavenベースで書かれている。Gradleもマニュアルにはあるがプラグインのことなどはあまり詳細が書いてない。

# チュートリアルのサンプル

付属のマニュアルのチュートリアルで書かれているサンプルを元に作成する。なおデータベースはチュートリアルにあるMySQLではなくSQLiteを使った。

## Step 1 準備
必要なライブラリを設定する。

```build.gradle
apply plugin: 'java'
// jooqのGradleプラグインと
apply plugin: 'nu.studer.jooq'

repositories {
    mavenCentral()
}

buildscript {
    repositories {
        jcenter()
        mavenCentral()
    }

    dependencies {
        // jooqのGradleプラグイン
        classpath 'nu.studer:gradle-jooq-plugin:1.0.5'
        // SQLiteを使う
        classpath group: 'org.xerial', name: 'sqlite-jdbc', version: '3.8.11.2'
    }
}

dependencies {
    compile group: 'org.jooq', name:'jooq', version: '3.8.3'
    compile group: 'org.jooq', name:'jooq-meta', version: '3.8.3'
    runtime group: 'org.xerial', name: 'sqlite-jdbc', version: '3.8.11.2'
}
```

## ステップ 2 データベース
データベースを作成

```
$ sqlite library.db
sqlite> CREATE TABLE `author` (
  `id` int NOT NULL,
  `first_name` varchar(255) DEFAULT NULL,
  `last_name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
);
sqlite> INSERT INTO "author" VALUES(1,'Hoge','Foo');
sqlite> INSERT INTO "author" VALUES(2,'Bar','Baz');
```
こんな感じでデータも入れておく。

## ステップ 3 コード生成
次の設定を追加。この書式はGradleプラグインのおかげである。プラグイン無しだとちょっと設定が面倒。

```build.gradle

jooq {
    library (sourceSets.main) {
        jdbc {
            driver = "org.sqlite.JDBC"
            url = 'jdbc:sqlite:./library.db'
        }
        generator {
            target {
                packageName = 'example.jooq'
            }
        }
    }
}
```

```
$ gradle  generateLibraryJooqSchemaSource
```

gradleのターゲットの名前は設定(library)によって変わってくる。

## ステップ 4,5,6 
ここは同じなのでまとめた。

```Main.java
import org.jooq.DSLContext;
import org.jooq.Record;
import org.jooq.Result;
import org.jooq.SQLDialect;
import org.jooq.impl.DSL;

import java.sql.Connection;
import java.sql.DriverManager;

import static example.jooq.Tables.AUTHOR;


public class Main {
    public static void main(final String[] args) {
        final String userName = "";
        final String password = "";
        final String url = "jdbc:sqlite:library.db";

        try (Connection conn = DriverManager
                .getConnection(url, userName, password);
             final DSLContext create = DSL.using(conn, SQLDialect.MYSQL)) {
            final Result<Record> result = create.select().from(AUTHOR).fetch();

            for (final Record r : result) {
                final Integer id = r.getValue(AUTHOR.ID);
                final String firstName = r.getValue(AUTHOR.FIRST_NAME);
                final String lastName = r.getValue(AUTHOR.LAST_NAME);

                System.out.println(
                        "ID: " + id + " first name: " + firstName + " last name: " + lastName);
            }
        } catch (final Exception e) {
            e.printStackTrace();
        }
    }
}
```

SQLiteのファイルパスが違うとテーブルが見つからないという一見分かりにくいエラーを出すので注意。



# 参考
* http://qiita.com/rubytomato@github/items/93ed18b30605f4491086
* http://www.jooq.org/doc/3.8/manual-single-page/#tutorials
* https://github.com/etiennestuder/gradle-jooq-plugin - このプラグインを使った

