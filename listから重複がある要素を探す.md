色々方法あるんだろうけど。

```python3
>>> from collections import Counter
>>> [ k for k, v in Counter([0,1,2,3,2,0,4]).items() if v > 1 ]
[0, 2]
>>> [ k for k, v in Counter("abcefacc").items() if v > 1 ]
['c', 'a']
```
