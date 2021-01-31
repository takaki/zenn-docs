---
emoji: "💻"
type: "tech"
published: true
title: twythonで実装されてないAPIを使う
topics: ["Python","Twitter"]
author: takaki@github
slide: false
---
PythonでtwitterのAPIを使おうとtwythonを使ったらunretweetが実装されてなかった。ということで内部実装を使いましたという例。


```py3
from twython import Twython

twitter = Twython(APP_KEY, APP_SECRET, ACCESS_TOKEN, ACCESS_TOKEN_SECRET)
# twitter.unretweet(id=ID) は無い
twitter.post('statuses/unretweet/%s' % ID)
```

あとはTwitterのAPIリファレンスを参考に。

