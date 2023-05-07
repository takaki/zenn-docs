---
title: "JetBrainsでAWS CodeWhispererを使う"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS, CodeWhisperer]
published: true
---
チュートリアルのビデオを見るのが一番早いが一応説明する。

https://aws.amazon.com/jp/codewhisperer/resources/#Getting_started

個人使用で説明する。この場合はまずAWS Builder IDを取得する必要がある。

https://profile.aws.amazon.com/

メールアドレスを入力して順次やっていく。

IntelliJか何かしらJetBrainsのIDEを起動する。次に設定のプラグイン追加でAWS Toolkitをインストールする。

サイドバーのAWS Toolkitを開く。自分はここの気付かなかった。

Developer ToolタブにCodeWhispererが表示される。ここからCodeWhispererを起動させる。
初回はAWS Builder IDでのログインが求められるのでブラウザでログイン処理を行う。

あとは実際に試す。

実際Rustで以下まで入力して補完をしていったら
```rust
// add two integers
fn
```

以下のような想定通りのコードが出てきたのでまずまずか。

```rust
// add two integers
fn add(a: i32, b: i32) -> i32 {
    a + b
}
```