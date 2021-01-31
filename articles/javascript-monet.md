---
emoji: "💻"
type: "tech"
published: true
title: JavaScriptの関数型言語風のライブラリmonet
topics: ["JavaScript","関数型言語"]
author: takaki@github
slide: false
---
undefinedとnullの扱いがめんどくさくてMaybeはできないのかと探したらmonetというライブラリがあった。
FunctionalJavaやScalazを参考に開発しているようでかなり使いやすそう。Maybe・Either・IO・Listなどなどがあるが今回はMaybeさえ使えればよかったのでこだけ紹介する。

インストールはnpmでいつものように。

```
$ npm install --save monet
```

次のような感じで使う。

```js
import monet from 'monet'

const m = monet.Maybe.fromNull(x)
const a = m.map( o => doSomething(o)).orJust(foo)
```

他の言語でMaybeやOptionalを使ったことがあればすぐにわかるだろう。


* https://cwmyers.github.io/monet.js/

