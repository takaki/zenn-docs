---
title: "Slack APIをPython SDKで使う"
emoji: "😽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Slack", "Python"]
published: true
---

Python Slack SDKを使ってSlackにメッセージを送ってみた。Slack Appに触るの久しぶりなのでいろいろ変わってた。

## 準備： Slack Appを登録する

詳細な説明は省きます。

- <https://api.slack.com/apps> からCreate New Appから作成
- OAuth & Permission に移動 -> Scopeから Bot Token Scopeを追加
- chat:write.public を追加。これでメンバーでないchannelにも書けるようになる
- 上のほうのinstall to workspace で使えるようになる
- Bot User OAuth Access Tokenをコピーして`.env`などに記入しておく
  - 使うAPIによってはOAuth Access Tokenが必要

## Slack APIのクライアントを作る

- slack_sdk をインストール <https://pypi.org/project/slack-sdk/>

```py
import os

from slack_sdk import WebClient
from dotenv import load_dotenv

load_dotenv(verbose=True)

token = os.environ.get("SLACK_BOT_TOKEN")
client = WebClient(token=token)
# APIによっては使い分け
# token = os.environ.get("SLACK_OAUTH_TOKEN")
# client = WebClient(token=token)
```

## 通常のメッセージを送る

```py
client.chat_postMessage(channel="#devtest", text="Hello world")
```

## 特定のユーザーにメンションする

`@`をそのままは使えない。`<@UXXXXXXXX>` という形式で`UXXXXXXX` はSlackのUser ID。
User IDはSlackのプロフィールから表示できる。

```py
client.chat_postMessage(channel="#devtest", text="<@UXXXXXXX1> Hello world")
```

## グループに送る

groupは<!subteam^groupId> という形式。groupIdはweb クライアントだとURLからわかる。

## スレッドに返信する

返信対象にするメッセージをパラメータで指定する必要がある。メッセージを送ったときのレスポンスから取得できる。

```py
response = client.chat_postMessage(channel="#devtest", text="begin thread")
client.chat_postMessage(channel="#devtest", text="reply", thread_ts=response['message']['ts'])
# reply_boradcastをつけるとチャンネルにも投稿される
client.chat_postMessage(channel="#devtest", text="reply 2", thread_ts=response['message']['ts'], reply_broadcast=True)
```

## @channel に送る

```py
client.chat_postMessage(channel="#devtest", text="<!channel> Hello world")
```

## 他

- <https://gitlab.com/-/snippets/2076762> 他いろいろサンプル