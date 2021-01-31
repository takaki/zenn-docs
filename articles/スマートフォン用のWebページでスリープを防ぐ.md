あるウェブページを表示させているときにスマートフォンの画面をスリープになるのをJavaScriptだけで防ぎたかったので探してみた。NoSleep.jsというライブラリを発見した。

インストールは次の通り。

```
$ npm install --save nosleep.js
```

以下サンプル。サンプルを作るのにめんどくさかったのでReactを使ってあるが本質ではない。

```js
import NoSleep from 'nosleep.js/NoSleep'

class App extends Component {
    constructor() {
        super();
        this.noSleep = new NoSleep();

        this.state = {sleep: "Sleep"}
    }

    render() {
        return (
            <div>
                <button onClick={() => {
                    this.setState({sleep: "No Sleep"});
                    this.noSleep.enable();
                }}>No Sleep
                </button>
                <button onClick={() => {
                    this.setState({sleep: "Sleep"});
                    return this.noSleep.disable();
                }}>Sleep
                </button>
                {this.state.sleep}
            </div>
        );
    }
}
```

NoSleepのインスタンスを作りボタンが押されあときのイベントハンドラでenableを実行する。このenableはイベントハンドラの中で実行する必要がある。コンストラクタの中で実行しても有効にならない。

実装はJavaScriptに埋め込まれた見えない小さな動画を再生することで画面がスリープになるのを防いでいるようだ。

## 参考
* https://github.com/richtr/NoSleep.js
