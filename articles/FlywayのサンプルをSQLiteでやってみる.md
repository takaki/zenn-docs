FlywayはJDBCを使ったデータベースマイグレーションツール。FlywayのページにはH2+Gradleのサンプルが書いてあるがSQLiteでやってみた。

```build.gradle
apply plugin: 'java'
apply plugin: "org.flywaydb.flyway"

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath group: "org.flywaydb", name: "flyway-gradle-plugin", version: "4.0.3"
        classpath group: 'org.xerial', name: 'sqlite-jdbc', version: '3.8.11.2'
    }
}

repositories {
    mavenCentral()
}

flyway {
    url = "jdbc:sqlite:./target/sample.db"
}
```

flywayのpluginを導入するがpluginの定義に注意。Flywayのサイトにはplugins{}を使ってあるがエラーが発生した。原因を調べるのも面倒なのでGradleのドキュメントに説明してある方法でpluginを定義してある。

次にmigrationのSQLのファイルを作成する。

```src/main/resources/db/migration/V1__Create_person_table.sql
create table PERSON (
    ID int not null,
    NAME varchar(100) not null
);
```

このあとにmigrationを実行する。

```
$ gradle flywayMigrate 
```

テーブルの作成を確認する。

```
$ sqlite3 target/sample.db '.schema PERSON'
CREATE TABLE PERSON (
    ID int not null,
    NAME varchar(100) not null
);
```

次にデータを挿入するファイルを作成する。

```main/resources/db/migration/V2__Add_people.sql
create table PERSON (
    ID int not null,
    NAME varchar(100) not null
);
```

このあと同様にmigrationを実行する。

```
$ gradle flywayMigrate 
```

テーブルの中身を確認する。

```
$ sqlite3 target/sample.db 'select * from person'
1|Axel
2|Mr. Foo
3|Ms. Bar
```

基本的に何も変わらない。H2をごちゃごちゃするよりSQLiteのほうが楽だったので試してみた。

# 参考
* https://flywaydb.org/getstarted/firststeps/gradle
