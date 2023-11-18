---
title: "KottiのデータをSeaORMで取り出す"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Rust, SeaORM]
published: true
---
Kotti (https://github.com/Kotti/Kotti) というPythonによるCMSを以前使っていたが使わなくなったのでデータだけを取り出したくなった。
データはSQLiteで保存していた。これをRustで取り出すこととして，SeaORMを使ってみることにした。
今回はSeaORM CLIというツールで既存のテーブルからコードを生成した。


### コード生成

まずは SeaORM CLIのインストール。

```shell
cargo install sea-orm-cli
```

`cargo init`でプロジェクトを作るあたりは省略。

`Cargo.toml`で`sea-orm`を使うように指定。featuresでいくつか必要なものを追加。
```toml
[dependencies]
sea-orm = { version = "0.12.6", features = ["sqlx-sqlite", "runtime-tokio-rustls", "macros"] }
tokio = { version = "1.34.0", features = ["full"] }
```

sea-orm-cliを実行する。以下にようにすれば`src/entities`以下にコードが生成される。

```shell
sea-orm-cli generate entity -u sqlite:///path/to/Kotti.db -o src/entities
```

`main.rs`を雑に書いてどんなものか様子を見ようとする。

```shell
use std::error::Error;

mod entities;

#[tokio::main]
async fn main() -> Result<(), Box<dyn Error>> {
    Ok(())
}

```

これでOK…とはならない。実際`cargo build`を実行すると以下のようなエラーになる。
```shell
error: Entity must have a primary key column. See <https://github.com/SeaQL/sea-orm/issues/485> for details.
 --> src/entities/kotti_alembic_version.rs:5:35
  |
5 | #[derive(Clone, Debug, PartialEq, DeriveEntityModel, Eq)]
  |                                   ^^^^^^^^^^^^^^^^^
  |
  = note: this error originates in the derive macro `DeriveEntityModel` (in Nightly builds, run with -Z macro-backtrace **for more info)
```

SeaORMはPrimary Keyがないテーブルにまだ対応してないらしい。
そこをなんとかする…ということは今回はやらない。
このテーブルのデータは必要ないのでファイル毎消してしまうことにする。
するとコンパイルは通るようになる。

ちょっとデータベースから実際にデータを取ってくる。

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn Error>> {
    let url = "sqlite:///home/takaki/var/Kotti.db";
    let conn = Database::connect(url).await?;

    let node = Nodes::find_by_id(1).one(&conn).await?.unwrap();
    println!("{:?}", node);
    let nodes = Nodes::find().all(&conn).await?;
    nodes.iter().for_each(|n| {
        println!("{:?}", n);
    });

    Ok(())
}
```

こんな感じであとは頑張る。