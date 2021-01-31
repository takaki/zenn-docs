とあるPythonスクリプトを動かす環境のLANGがja_JP.UTF-8からCに変わったらエラーが出るようになったのでそのあたりの対処をメモ。

```py3
#!/usr/bin/python3
# -*- coding: utf8 -*-

import sys

from logging import getLogger, StreamHandler, DEBUG

handler = StreamHandler()

logger = getLogger(__name__)
logger.setLevel(DEBUG)
logger.addHandler(handler)


str = "ほげ"
tf = open('utf8.txt')
s = tf.read()

print(str)
print(s)

logger.debug(str)
logger.debug(s)
```
これをLANG=ja_JP.UTF-8で動かすのは大抵問題がない。これは大抵起動時にLANGを見て入出力のデフォルトエンコーディングを変更するようにPythonが設定されているからである。なのでこれをLANG=Cに持っていくとUTF-8をASCIIとして解釈しようとして例外を出すようになる。

まずファイルの読み込みにエンコーディングを指定する。

```py3
tf = open('utf8.txt', encoding='utf8')
```

printで使う sys.stdout の出力先がUTF-8であると指定する。

```py
sys.stdout = codecs.getwriter("utf8")(sys.stdout.detach())
sys.stderr = codecs.getwriter("utf8")(sys.stderr.detach())
```

loggerはこのままでも動くがエスケープされた出力となってしまうので変更したsys.stderrを使うようにする。

```py
handler = StreamHandler(sys.stderr)
```

まとめるとこうなる。

```py
#!/usr/bin/python3
# -*- coding: utf8 -*-

import codecs
import locale
import sys


from logging import getLogger, StreamHandler, DEBUG, Formatter

sys.stdout = codecs.getwriter("utf8")(sys.stdout.detach())
sys.stderr = codecs.getwriter("utf8")(sys.stderr.detach())

handler = StreamHandler(sys.stdout)

logger = getLogger(__name__)
logger.setLevel(DEBUG)
logger.addHandler(handler)



str = "ほげ"
tf = open('utf8.txt', encoding='utf8')
s = tf.read()

print(str)
print(s)

logger.debug(str)
logger.debug(s)
```
# 参考
* http://stackoverflow.com/questions/4374455/how-to-set-sys-stdout-encoding-in-python-3
