---
emoji: "💻"
type: "tech"
published: true
title: react-reduxの使い方
topics: ["JavaScript","React","flux","redux"]
author: takaki@github
slide: false
---
内容が古いので[Reactコンポーネントに状態を持たせる方法の比較](https://qiita.com/takaki@github/items/89f788af75482956e7dd)を見て下さい。

---

前提としてReactを使ったページを作成のための知識はあるものとする。またFluxについての解説はしない。そのあたりは [webpack+React+material-uiの環境を最小手順で作成](http://qiita.com/takaki@github/items/724d97a20d3ae194ded4) を参考。

次のサンプルを使って話をすすめる。ボタンを押すとカウントするよくあるサンプルである。

```javascript:entry.js
import  React from 'react';
import  ReactDOM from 'react-dom';

var Counter = React.createClass({
    render: function(){
        return (
            <h1>Count = {this.props.count}</h1>
        )
    }
})

var Plus1 = React.createClass({
    render: function() {
        return(<button onClick={() => this.props.onClick(1)}>plus 1</button>)
    }
})

var PlusForm = React.createClass({
    getInitialState: function() {
        return {
            num: 0
        }
    },
    render: function() {
        return (
            <div>
                <div>
                    <button onClick={() => this.props.onClick(this.state.num)}>plus {this.state.num}</button>
                    <input type="text" onChange={(ev)=>this.setState({num: parseInt(ev.target.value)||0})}/>
                </div>

            </div>
        )

    }
})

var Reset = React.createClass({
    render: function() {
        return (
            <div><button onClick={this.props.onClick}>Reset</button></div>
        )
    }

})

var App = React.createClass({
    getInitialState: function() {
        return {
            count: 0
        }
    },
    addCounter: function(n) {
        this.setState({count: this.state.count + n})
    },
    render: function() {
        return (
            <div>
                <Counter count={this.state.count}/>
                <Plus1  onClick={() => this.addCounter(1)}/>
                <PlusForm onClick={(n) => this.addCounter(n)}/>
                <Reset onClick={() => this.setState({count: 0})}/>
            </div>
        )
    }
})

ReactDOM.render(
    <App/>,
    document.getElementById('content')
);

```

## react-redux の導入

これをreduxに対応するように書き換える。

まずreduxとreact-reduxを導入する。

```
npm install --save redux react-redux
```
インストールしたのちにインポートを行う。

```javascript
import { createStore } from 'redux'
import { connect, Provider } from 'react-redux'
```

stateとその操作を行うコードを作成する。

```javascript
// ここで返されるオブジェクトが状態の変更を表すアクションになる
// type が操作の内容 payload がパラメータ
// reduxのアクションはどう書いても良いのだが FSA という標準が提案されている。
function addCounter(num) {
    return {
        type: 'ADD_COUNTER',
        payload: num
    }
}

function resetCounter(num) {
    return {
        type: 'RESET_COUNTER',
    }
}

function setButton(num) {
    return {
        type: 'SET_BUTTON',
        payload: num
    }
}

// 初期状態
const initialState =  {
    count: 0,
    button: 0,
}

const reducer = function(state = initialState, action) {
    // actionに応じて状態をどう変更するか記述する
    // 必ず元の state を元に新しい state を作成して返す形になる
    switch(action.type) {
        case "ADD_COUNTER":
            return Object.assign({}, state, {
                count: state.count + action.payload
            })
        case "RESET_COUNTER":
            return Object.assign({}, state, {
                count: 0
            })
        case 'SET_BUTTON':
            return Object.assign({}, state, {
                button: action.payload,
            })
        default:
            return state;
    }
}

// store が状態を管理するオブジェクトになる
const store = createStore(reducer);
```

* FSA: https://github.com/acdlite/flux-standard-action

次にReactのオブジェクトと Redux のオブジェクトを継ぐ。

```js

// storeが管理するstateを props として受け取るための変換函数
function mapStateToProps(state, props) {
    return state
}

// 各コンポーネントのイベントハンドラを一括で作成するものと思えば良い
// これも props に割り当てられる
function mapDispatchToProps(dispatch, props) {
    return {
        addCounter: function(n) {
            dispatch(addCounter(n));
        },
        resetCounter: function() {
            dispatch(resetCounter());
        },
        setButton: function(n) {
            dispatch(setButton(n));
        }
    }

}

// connect函数で元のReactのコンポーネントをreduxに対応したものに変換する
// 元の変数に代入しているのが気持ち悪ければ適当に変更する
App = connect(mapStateToProps,mapDispatchToProps)(App)

// おまじない的なもの。Providerで受け取ったstoreを渡す
ReactDOM.render(
    <Provider store={store}>
        <App/>
    </Provider>,
    document.getElementById('content')
);
```

次に各コンポーネントを書き換える。

```js
var App = React.createClass({
// コンポーネントで管理していた状態は削除する
// コンポーネントに属していたイベントハンドラも削除
// this.state.countで取得していたものを this.props.countで取得できるようする
// イベントハンドラもpropsから取得
    render: function() {
        return (
            <div>
                <Counter count={this.props.count}/>
                <Plus1 onClick={this.props.addCounter}/>
                <PlusForm button={this.props.button}
                     onClick={this.props.addCounter}
                     onChange={this.props.setButton}/>
                <Reset onClick={this.props.resetCounter} />
            </div>
        )
    }
})
```

次に子コンポーネントも書き換え

```js
var PlusForm = React.createClass({
// 同様にstate とイベントハンドラをプロパティ経由で受け取る
    render: function() {
        return (
            <div>
                <div>
                    <button onClick={() => this.props.onClick(this.props.button)}>plus {this.props.button}</button>
                    <input type="text" onChange={(ev)=> this.props.onChange(parseInt(ev.target.value)||0)}/>
                </div>

            </div>
        )

    }
})
```

以上でreact-reduxに対応するように書き換えた。

## redux-actions

reduxで書いていると冗長な部分がある。これを簡素に書き換えるための便利ツールが redux-actions である。

導入はnpm でいつものように。

```bash
npm install --save redux-actions
```

次に jsファイルに追加

```js
import { createAction, handleActions } from 'redux-actions';
```

createAction を使うと以下のような書き換えができる。

```js
// function addCounter(num) {
//    return {
//       type: 'ADD_COUNTER',
//        payload: num
//    }
// }

var addCounter = createAction('ADD_COUNTER');
```

reducerも以下のように簡潔になる

```js

const reducer = handleActions({
    [addCounter]: (state, action) => Object.assign({}, state, {
        count: state.count + action.payload
    }),
    [resetCounter]: (state, action) => Object.assign({}, state, {
        count: 0
    }),
    [setButton]: (state, action) => Object.assign({}, state, {
        button: action.payload,
    })},
    initialState
);
```
## 参考
* https://github.com/takaki/demo-react-redux 今回使ったサンプルプログラム
* https://github.com/acdlite/redux-actions
* https://github.com/acdlite/flux-standard-action

