---
emoji: "💻"
type: "tech"
published: true
title: Reactでinputを扱う
topics: ["JavaScript","React"]
author: takaki@github
slide: false
---
[Reactでinputを扱う(TypeScript版)](https://qiita.com/takaki@github/items/9ac4e3f1e461fdaf7c30)という改訂版を書きました。

---

input要素をReactで使うとReactがinputを自分でコントロールするので注意が必要。

```js
var React = require('react');
var ReactDOM = require('react-dom');

var App = React.createClass({
    render: function() {
        return (
            <div>
                <label>value a</label>
                <input type="radio" name="aradio" value="A" checked="checked" /> <br />
                <label>value b</label>
                <input type="radio" name="aradio" value="B" />
                <hr />
                <label>text input</label>
                <input type="text" name="atext" value="" />
            </div>
        )
    }
});
```
このようにするとラジオボタンの操作はできないしテキストの入力もできない。input要素のvalue属性はstateを通して操作する必要がある。

```js
var App = React.createClass({
    getInitialState: function() {
        return {
            radio: "a",
            text: ""
        }
    },
    render: function() {
        return (
            <div>
                <label>value a</label>
                <input type="radio" name="aradio" value="A" checked={this.state.radio === 'a'}
                       onChange={() => this.setState({radio: 'a'})}/> <br />
                <label>value b</label>
                <input type="radio" name="aradio" value="B" checked={this.state.radio === 'b'}
                       onChange={() => this.setState({radio: 'b'})}/> 
                <hr />
                <label>text input</label>
                <input type="text" name="atext" value={this.state.text}
                       onChange={(e) => this.setState({text: e.target.value})}/>
            </div>
        )
    }
});
```
このように`onChange`においてハンドラを記述し，その中で`this.setState`を使ってthis.stateを書き直す形になる。

