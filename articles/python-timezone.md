---
emoji: "💻"
type: "tech"
published: true
title: Pythonのタイムゾーンの扱い
topics: ["Python"]
author: takaki@github
slide: false
---
環境: Python-3.5

Pythonの時刻はタイムゾーンを意識して作られている。タイムゾーンの扱いに関してのメモ。
timezone-naiveがタイムゾーンを意識してない時刻，timezone-awareがタイムゾーンを意識した時刻となる。

```python
from datetime import datetime,timezone
import  pytz

naive = datetime.now()
aware = datetime.now(timezone.utc)
```
`timezone#now`の引数に`timezone`を渡すとtimezone-awareな現在時刻が作られる。
この二つは比較できない。

```python
naive > awre 
# TypeError: can't compare offset-naive and offset-aware datetimes
```
例外がスローされる。

任意の時刻を作るときには以下のようにしてtimezoneを引数に渡す。

```python
naive = datetime(2016,6,1,12,0,0,0)
aware = datetime(2016,6,1,12,0,0,0,timezone.utc)
```

ローカルのタイムゾーンに変更したいときには`datetime#astimezone`を使う。
当然timezone-naiveな時刻は使えない。

```
print(naive.astimezone())
# ValueError: astimezone() cannot be applied to a naive datetime
print(aware.astimezone())
# 2016-06-01 21:00:00+09:00
```

自分の好きなタイムゾーンを使うときには`pytz`を使うと便利。
timezone-naiveな時刻を指定したタイムゾーンに変更する。

```python
import  pytz

de = pytz.timezone('Europe/Berlin')
print(de.localize(naive))
# 2016-06-01 12:00:00+02:00

jp = pytz.timezone('Asia/Tokyo')
print(jp.localize(naive))
# 2016-06-01 12:00:00+09:00
```

timezone-awareな時刻のタイムゾーンを変更する。
これは実際の時刻が変更されるわけではない。

```python
print(aware.astimezone(jp))
# 2016-06-01 21:00:00+09:00
print(aware.astimezone(de))
# 2016-06-01 14:00:00+02:00
```

`datetime#replace`でタイムゾーンを上書きできる。当然表現する時刻が変わってしまう。

```python
print(aware.replace(tzinfo=jp).astimezone(timezone.utc))
# 2016-06-01 03:00:00+00:00
print(aware.replace(tzinfo=de).astimezone(timezone.utc))
# 2016-06-01 11:07:00+00:00
```

