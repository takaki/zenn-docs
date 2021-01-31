---
emoji: "💻"
type: "tech"
published: true
title: redux-promiseの使い方
topics: ["JavaScript","React","redux"]
author: takaki@github
slide: false
---
この記事は古くなりました。redux-promiseは不要です。
redux で非同期アクションを使う (https://qiita.com/takaki@github/items/737630f112208832a1fd)
を読んで下さい。

---

# 前提
* React
* react-redux
* react-actions
* Promise

Redux で Action を処理に非同期処理，ajax を使おうとすると書き方が少しめんどくさい。 redux-promise というミドルウェアを使うときれいに書けるのだが README にまともに使い方の説明が書いてない。
少し解説をする。

# サンプル
```js
import  React from 'react';
import  ReactDOM from 'react-dom';
import { createStore, combineReducers, applyMiddleware } from 'redux'
import { Provider } from 'react-redux'
import { connect } from 'react-redux'
import { createAction, handleAction, handleActions } from 'redux-actions';
import promiseMiddleware from 'redux-promise';
import axios from 'axios';

const App = React.createClass({
    getLogin: function() {
    },
    render: function() {
        return (
            <div>
                <h1>name = {this.props.login}</h1>
                <button onClick={this.props.setLogin}>Get</button>
                <button onClick={this.props.reset}>Reset</button>
            </div>
        )
    }
})

const reset = createAction('RESET');
const setLogin = createAction('SET_LOGIN');

const reducer = handleActions({
    [setLogin]: (state, action) => Object.assign({}, state, {
        login: action.payload
    }),
    [reset]: (state, action) => Object.assign({}, state, {
        login: "reset"
    }),
}, {
    login: "empty",
});

const store = createStore(reducer);

const mapStateToProps = (state, props) => state;

const mapDispatchToProps = (dispatch, props) => {
    return {
        setLogin: function() {
            axios.get("https://api.github.com/users/takaki")
                 .then(function(response) {
                dispatch(setLogin(response.data.name));
            });
        },
        reset: () => dispatch(reset())
    }
};

const RApp = connect(mapStateToProps,mapDispatchToProps)(App)

ReactDOM.render(
    <Provider store={store}>
        <RApp />
    </Provider>,
    document.getElementById('content')
);
```

github の API を利用して ajax でデータを取ってきて表示するサンプルである。
mapDispatchtoProps の setLogin の定義中で axios を使い then の中でdispatchを呼び出している。この中でごちゃごちゃ書くよりaction の定義に押し込めたい。

```js
const setLogin = createAction('SET_LOGIN', () => 
            axios.get("https://api.github.com/users/takaki")
                 .then(function(response) {
                return response.data.name;
            });
```
と書ければいいのだがこのままだとPromiseが処理できないのでエラーになる。

# redux-promiseの導入
これをきちんと動くように処理してくれるミドルウェアがredux-promiseである。
インストールはnpmでいつものように。

```
npm install --save redux-promise
```

createStore を以下のようにすると promiseMiddleware を使用するようになる。

```
import promiseMiddleware from 'redux-promise';

#...

const store = createStore(reducer, applyMiddleware(promiseMiddleware));
```

ここで先程のようにsetLogin を書き換える。

```js
const setLogin = createAction('SET_LOGIN', () =>
    axios.get("https://api.github.com/users/takaki").then(function(response) {
        return response.data.name;
    })
);
```

次にディスパッチも書き換える。

```js
const mapDispatchToProps = (dispatch, props) => {
    return {
        setLogin: function() {
            dispatch(setLogin());
        },
        reset: () => dispatch(reset())
    }
};
```
このように自然な形で書けるようになる。

# 参考
* https://github.com/acdlite/redux-promise

