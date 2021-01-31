浮動小数点を少数位以下何桁で丸めたいとする。ちょっと検索すると次のようなコードを使えという記事がごろごろしている。

```js
Math.round(f*100)/100
Math.ceil(f*100)/100
Math.floor(f*100)/100
```

それぞれ四捨五入だの切り上げだの切り捨てだのという話はあるが Number.toFixed の話に触れてない記事が多い。そんな最近実装された機能だろうかと疑問に思ったがECMA 3rd には入っていたので相当昔である。toFixedの丸め方は厳密ではないにしろずらずら長い数値を表示したくないというだけなら十分である(他の言語のsprintfでできるレベルで良いという話)。

* https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Number/toFixed
