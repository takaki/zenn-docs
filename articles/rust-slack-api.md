---
title: "RustでSlack APIを使う"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rust", "Slack"]
published: true
---

RustでSlackのAPIを使ってみた。Slack Appの登録の方法などは省略。
Slack APIを他の言語で使った経験があり雰囲気はわかっているものとしている。

## cargoの設定

async clientを使うときにmainでtokio-1.0を使っているとslack-apiがtokio-0.2のためそのままだと動かない。
互換性のライブラリを入れる必要がある。さらにasyncを呼び出すときにcompat()をつける必要がある。

```Cargo.toml
tokio-compat-02 = "0.2.0"
```

sync clientを使うときには少し変更が必要。

```Cargo.toml
slack_api = {version="0.23.1", features = ["sync", "reqwest_blocking"]}
```

とオプションを追加する必要がある。

## async clientの例

Slack APIを他の言語で使った経験があれば大体わかるだろう。

```rs
use std::env;
use dotenv::dotenv;
use slack_api::chat::PostMessageRequest;
use tokio_compat_02::FutureExt;
use futures::TryFutureExt;

#[tokio::main]
async fn main() {
    dotenv().unwrap();
    let token = env::var("SLACK_BOT_TOKEN").unwrap();
    let client = slack_api::default_client().unwrap();
    let response = slack_api::chat::post_message(&client, &token, &PostMessageRequest {
        channel: "#devtest",
        text: "Hello Rust",
        ..PostMessageRequest::default()
    }).compat().await.unwrap();

    let ts = response.ts.unwrap();


    let response = slack_api::chat::post_message(&client, &token, &PostMessageRequest {
        channel: "#devtest",
        text: "response",
        thread_ts: Some(ts),
        ..PostMessageRequest::default()
    }).compat().await.unwrap();

    println!("{:?}", response);
}
```

- <https://gitlab.com/-/snippets/2076812> syncはこちらのスニペット参照