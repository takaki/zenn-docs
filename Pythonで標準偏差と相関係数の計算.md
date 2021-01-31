標準偏差と相関係数をいくつかの方法で計算してみた。

```py3
Python 3.5.2rc1 (default, Jun 13 2016, 09:33:26) 
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> x = [1, 2, 3, 4, 5]
>>> y = [0, 2, 4, 5, 8]
```
statisticsライブラリを使う。Python-3.4(?)から組み込み。

```py3
>>> import statistics
>>> statistics.pstdev(x), statistics.stdev(x)
(1.4142135623730951, 1.5811388300841898)
```
numpy

```py3
>>> import numpy
>>> numpy.std(x), numpy.std(x, ddof=1)
(1.4142135623730951, 1.5811388300841898)
```
scipyはnumpyと同じ?

```py3
>>> import scipy
>>> scipy.std(x), scipy.std(x, ddof=1)
(1.4142135623730951, 1.5811388300841898)
```
相関係数をnumpyで計算。

```py3
>>> numpy.corrcoef(x, y)[0, 1]
0.99044346677110506
```
次にscipy。

```py3
>>> import scipy.stats
>>> scipy.stats.pearsonr(x, y)
(0.99044346677110517, 0.0011198526620164956)
```

scipyが一番便利かな。
