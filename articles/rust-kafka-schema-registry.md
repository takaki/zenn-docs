---
title: "RustでKafkaのSchema Registryを利用する"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [rust, kafka]
published: true
---
RustでConfluentのKSQLの結果をconsumeするコードを作ってみた。

コード見ればわかる人は以下参照。

https://gitlab.com/-/snippets/2284040

kakfa と schema_registry_converter のサンプルを組み合わせたものになる。

https://crates.io/crates/kafka

https://crates.io/crates/schema_registry_converter

schema_registry_converterのサンプルだとrdkafkaを使っていたのでをそこの部分を置き換えてみた。

前提としてconfluentのチュートリアルをやってデータが投入されているものとする。

https://developer.confluent.io/tutorials/create-session-windows/ksql.html





以下ざっと解説。

brokerとschema registryを指定する。

```rust
    let registry_url = "http://localhost:8081";
    let broker_url = "localhost:29092";
```

購読するトピックの指定。
```rust
    let topic = "clicks";
    // let topic = "IP_SESSIONS";
```

実験目的だとgroup_idを変更しないと実行しても前見たところからの続きになっ
て確認が面倒くさいので毎度変えるためのおまじない。


```rust
    let group_id = Uuid::new_v4().to_simple().to_string();
```

このあとschema registryを利用するデコーダの定義とconsumerの定義が続く。

consumerの作成にはbuilderパターンを使っているが詳しいオプションの意味はKafkaのAPIのオプションなのでKafka自体のドキュメントを参照。他の言語でやったことがあれば理解も早いはず。

```rust
    let mut decoder = AvroDecoder::new(SrSettings::new(String::from(registry_url)));

    let mut consumer = Consumer::from_hosts(vec![broker_url.to_owned()])
        .with_topic_partitions(topic.to_owned(), &[0])
        .with_fallback_offset(FetchOffset::Earliest)
        .with_group(group_id)
        .with_offset_storage(GroupOffsetStorage::Kafka)
        .create()
        .unwrap();

```

あとはデータが送られてきたらデコードを行う。

```rust
                if let Ok(result) = decoder.decode(Some(m.value)) {
                    // println!("{:?}", m);
                    println!("{:?}", result.value);
                }
```

実行すると以下のような出力が得られるはずである。


```
Record([("IP", Union(String("51.56.119.117"))), ("URL", Union(String("/etiam/justo/etiam/pretium/iaculis.xml"))), ("TIMESTAMP", Union(String("2019-07-18T10:00:00Z")))])
Record([("IP", Union(String("51.56.119.117"))), ("URL", Union(String("/nullam/orci/pede/venenatis.json"))), ("TIMESTAMP", Union(String("2019-07-18T10:01:00Z")))])
Record([("IP", Union(String("53.170.33.192"))), ("URL", Union(String("/mauris/morbi/non.jpg"))), ("TIMESTAMP", Union(String("2019-07-18T10:01:31Z")))])
Record([("IP", Union(String("51.56.119.117"))), ("URL", Union(String("/convallis/nunc/proin.jsp"))), ("TIMESTAMP", Union(String("2019-07-18T10:01:36Z")))])
...略
```

文字列としてはちゃんと取れていそうではあるがでは個々の要素を取りたい…となったときには別の工夫が必要になる。

`result.value`はavro_rs::types::Valueというenumになっている。地道にenumを分解してね…ってことになるがこれは別の話なのでやらない(がよくある話だとは思う)。
