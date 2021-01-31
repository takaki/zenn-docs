---
emoji: "💻"
type: "tech"
published: true
title: webpack+React+material-uiの環境を最小手順で作成
topics: ["webpack","React","JavaScript","material-ui","YARN"]
author: takaki@github
slide: false
---

create-react-app(https://github.com/facebook/create-react-app) を使って下さい。

```
$ yarn global add create-react-app
$ npx create-react-app project_name
```
で雛形ができます。

あとは

```
$ cd project_name
$ yarn add react @material-ui/core
```

などをやってください。


---
**以下の内容は古くなりました。**


表題通りで`React`と`matrial-ui`を使ったウェブページをwebpackを使って最小手順で作るための手順をまとめた。

# 準備
まずはディレクトリを作成して`npm`の準備をする。

```
$ mkdir webpack-exe
$ cd webpack-exe
$ npm init
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sane defaults.

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg> --save` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
name: (webpack-exe) 
version: (0.0.0) 
description: 
entry point: (index.js) entry.js
test command: 
git repository: 
keywords: 
author: 
license: (ISC) Public Domain
About to write to /home/takaki/src/webpack-exe/package.json:

{
  "name": "webpack-exe",
  "version": "0.0.0",
  "description": "",
  "main": "entry.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "Public Domain"
}


Is this ok? (yes) 

```

`webpack`がインストールされてない場合はインストールする。

```
$ npm install -g webpack
```
# Reactを動かす
次に`React`のインストール。

```
$ npm install --save react react-dom babel-preset-react babel-loader babel-core
$ npm install --save babel-preset-es2015
```
Reactで使うJSXはそのままではJavaScriptとして理解できないので変換するツールがbabel。
ES2015も使いたいのでbabelの対応パッケージもインストール。

次にHTMLとJavaScriptの準備。

```html:index.html
<!DOCTYPE html>
<html>

<head>
  <title>Webpack Excercise</title>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

</head>

<body>
  <div id="content"> Content </div>
  <script src="bundle.js" charset="utf-8"></script>
</body>

</html>
```
ちなみに scriptタグを入れる場所が重要で後ろのほうにしないとDOMの解釈のタイミングとかでエラーになるので注意。

```javascript:entry.js
var React = require('react');
var ReactDOM = require('react-dom');

ReactDOM.render(
  <h1>Hello, world</h1>,
  document.getElementById('content')
);
```
とする。

次にwebpackでビルドするための準備。
babelの設定ファイルを準備。

```javascript:.babelrc
{ "presets": ["react", "es2015"] }
```
これで React と ES2015 が使えるようになる。
次にwebpackの設定。

```javascript:webpack.config.js
module.exports = {
  entry: "./entry.js",
  output: {
    path: __dirname,
    filename: "bundle.js"
  },
  module: {
    loaders: [
      { test: /\.js$/,  loader: "babel-loader", exclude: /node_modules/ }
    ]
  }
}
```
entry.jsをコンパイルしてbundle.jsを生成する。jsファイルはbabelで扱う設定。

あとは webpack を動かすだけ

```
$ webpack --progress --color
```

と実行して index.html を見れば良い…ということなんだがwebpackの開発用webサーバーを使う。webサーバーが用意されてて便利というのもあるが差分コンパイルをしてくれるので早くなるという利点が大きい。

```
$ npm install webpack-dev-server -g
$ webpack-dev-server --progress --color

```

ここで http://localhost:8080/webpack-dev-server/ にアクセスすれば良い。


# material-ui のインストール

Reactが動くところを確認したらmaterial-uiのインストール。

```
$ npm install material-ui --save
```

次に entry.js を書き換える。

```javascript:entry.js
var React = require('react');
var ReactDOM = require('react-dom');

import MuiThemeProvider from 'material-ui/styles/MuiThemeProvider';
import RaisedButton from 'material-ui/RaisedButton';

ReactDOM.render(
  <div>
    <h1>Hello, world</h1>
    <MuiThemeProvider>
      <RaisedButton label="Default" />
    </MuiThemeProvider>
  </div>,
  document.getElementById('content')
);
```

webpack-dev-server を使っていると勝手にリロードされて画面が書き変わっているはず。

なおまとめたコードは https://github.com/takaki/webpack-exe を参照。

以上。

# 参考リンク
* http://webpack.github.io/docs/tutorials/getting-started/
* https://facebook.github.io/react/docs/package-management.html
* https://github.com/callemall/material-ui


