---
emoji: "💻"
type: "tech"
published: true
title: listから重複がある要素を探す
topics: ["Python"]
author: takaki@github
slide: false
---
色々方法あるんだろうけど。

```python3
>>> from collections import Counter
>>> [ k for k, v in Counter([0,1,2,3,2,0,4]).items() if v > 1 ]
[0, 2]
>>> [ k for k, v in Counter("abcefacc").items() if v > 1 ]
['c', 'a']
```

