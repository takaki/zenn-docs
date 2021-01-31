React Routerを使ってページを分割するときに各コンポーネントを動的インポートで使いたい場合は以下のようにする。

まず必要なライブラリの追加。

```
$ yarn add @loadable/component
```
デモとして次のようなファイルを作成する

```demo-import-async.tsx
import React from "react";

const now = new Date().toLocaleString();
const DemoImportAsync: React.FC = () => <div>{now}</div>;
export default DemoImportAsync;
```

```demo-import-sync.tsx
import React from "react";

const now = new Date().toLocaleString();
const DemoImportSync: React.FC = () => <div>{now}</div>;
export default DemoImportSync;
```
中身は実質同じだがこの二つのファイルを静的・動的の二つの方法で読み込む。
機能としては時刻を表示するだけである。

```App.tsx
// 他のimport省略
import DemoImportSync from "./demo-import-sync";
import loadable from "loadable-components";

// こうすることで動的インポートをするコンポーネントが作られる
const LoadableComponent = loadable(() => import("./demo-import-async"));

const App: React.FC = () => {
  return (
    <div className="App">
      <BrowserRouter>
        <div>
          <Link to="/import-sync">import-sync</Link>/
          <Link to="/import-async">import-async</Link>
        </div>
        <Switch>
          <Route path="/import-sync" exact={true}>
            <hr />
            <DemoImportSync />
          </Route>
          <Route path="/import-async" exact={true}>
            <hr />
            <LoadableComponent />
          </Route>
        </Switch>
      </BrowserRouter>
    </div>
  );
};

export default App;
```

このコンポーネントを表示させるとわかるが `/import-sync` のほうはページをロードした時刻が表示され `/import-async` のほうは実際に対象のURLにアクセスした時刻が表示される。

* https://reacttraining.com/react-router/web/guides/code-splitting
* https://github.com/smooth-code/loadable-components
