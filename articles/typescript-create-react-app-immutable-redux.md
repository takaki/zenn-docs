---
emoji: "💻"
type: "tech"
published: true
title: TypeScriptでcreate-react-appを使う(ImmutableとかReduxとかいろいろ含む)
topics: ["TypeScript","React","JavaScript"]
author: takaki@github
slide: false
---
create-react-appを使うとReactの雛形を作ってくれるがTypeScriptには対応していないし，当面対応する予定もない。しかしforkしてTypeScriptの対応をしてくれたcreate-react-app-typescriptがある。これを使ってJavaScriptで作ったアプリをTypeScriptに書き直した。調べるのに苦労したので作業メモもかねてImmutableやReduxなどなどを使うアプリのサンプルを説明する。

以下のものを使う。JavaScriptによるReactアプリ作成・TypeScriptなどの要素技術は理解しているものとする。

* yarn
* TypeScript-2.2.1
* React
* create-react-app
* Immutable
* Redux
* material-ui


# 初期設定

```
$ yarn global add create-react-app
$ create-react-app demo-react-scripts-ts --scripts-version=react-scripts-ts
```

# 他に必要なものをインストール
```
$ yarn add immutable
$ yarn add material-ui @types/material-ui
$ yarn add redux
$ yarn add redux-actions @types/redux-actions
$ yarn add react-redux @types/react-redux
$ yarn add react-tap-event-plugin
```

# App.tsxの開設
単純にボタンを押して数字を増やしたり減らしたりして表示する。

```ts
class Counter extends Record({counter: 0}) {
    counter: number;

    modify(diff: number) {
        return new Counter(this.set('counter', this.counter + diff));
    }
}
```

Immutableを使ってデータを保持するクラスを作成する。Record#setがMapに変わってしまうのでこういうやりかたになる。TypeScriptはJavaScriptより型がきちんとあっていいのだがimmutableなコレクションを使いにくい部分があるかなと思う。


```ts
interface AppProps {
    data: Counter;
    updateModel: (d: Counter) => void;
}

class App extends React.Component<AppProps, undefined> {
```

アプリのメインの記述。`Component<any,any>`としておけばなんでも通るがinterfaceを渡せばコンパイル時に変なpropsを書いてないかチェックできるようになる。2番目は`this.stateのinterfaceだがReduxを使うのでthis.stateを使うことはない。

```
    render() {
        return (
            <MuiThemeProvider>
                <div className="App">
                    <h1>{this.props.data.counter}</h1>
                    <FloatingActionButton onClick={() => this.props.updateModel(this.props.data.modify(1))}>
                        <ContentAdd/>
                    </FloatingActionButton>
```

`FloatingActionButton`の`onChange`の中身がモデルの更新作業となる。

```
                    <FloatingActionButton
                        secondary={true}
                        onClick={() => this.props.updateModel(this.props.data.modify(-1))}
                    >
                        <ContentRemove/>
                    </FloatingActionButton>
                </div>
            </MuiThemeProvider>
        );
    }
}
```


ここからReduxの記述になる。

```ts
const initialState = new Counter();
```

先程作ったモデルを初期値とする。


```ts
const updateModel = createAction<Counter>('UPDATE_MODEL');
```

redux-actionsで作るactionはモデルを更新するという一つだけで済む。実際のデータの更新はモデルクラスのメソッドを使う。

```ts
const reducer = handleActions<Counter>(
    {
        ['UPDATE_MODEL']: (state: Counter, action: Action<Counter>) => action.payload
    },
    initialState
);
```

更新作業も直感的な作業となる。

```ts
const store = createStore(reducer);

function mapStateToProps(state: Counter) {
    return {data: state};
}

function mapDispatchToProps(dispatch: Dispatch<Counter>) {
    return {
        updateModel: function (m: Counter) {
            dispatch(updateModel(m));
        }
    };

}

```

メソッドの型をきちんと入れなければならない。最初は型がわからなかった。

```
export default class RApp extends React.Component<undefined, undefined> {
    render() {
        const DApp = connect(mapStateToProps, mapDispatchToProps)(App);
        return (
            <Provider store={store}>
                <DApp />
            </Provider>);
    }
}
```

このようになる。

* https://github.com/takaki/demo-react-scripts-ts - githubに置いてあります

