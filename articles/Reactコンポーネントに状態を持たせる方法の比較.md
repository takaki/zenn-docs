Reactのコンポーネントに状態を持たせる方法をいくつかがあるが以下を紹介する。
* ReactのuseState
* React Redux
* ReactのuseReducer
* React ReduxのHooks

サンプルの内容は`input`でテキストを編集して`input`の中身を別の場所で表示する単純なもの。

## ReactのuseState

ReactのuseStateを使う。一番簡潔に書ける。一つのコンポーネント内で完結するのならば一番楽。

不便な点は複雑なコンポーネントになるとコールバックを子供の引数に渡してあちこち引き回すことになってややこしいこと。

```tsx
const UseState: React.FC = () => {
  const [text, setText] = useState("");
  return (
    <div>
      value={text}
      <input
        type="text"
        value={text}
        onChange={event => setText(event.target.value)}
      />
    </div>
  );
};
```


## React Redux

reduxを使った普通の書き方。グローバルに定義されたストアから状態を参照できる。状態の書き換えは`reducer`に集約される。お約束の定義がそれなりにあるのでこのサンプル程度のことをやるにはめんどくさい。あとコンポーネントのpropsをマッピングを定義するのがボイラープレートぽくて大量に書くと嫌になる。

```tsx
const SET_TEXT = "SET_TEXT";

interface SetTextAction {
  type: typeof SET_TEXT;
  payload: string;
}

type Action = SetTextAction;

interface State {
  text: string;
}

const reducer = (state: State = { text: "" }, action: Action): State => {
  switch (action.type) {
    case "SET_TEXT":
      return { text: action.payload };
    default:
      return state;
  }
};

const store = createStore<State, Action, {}, {}>(reducer);

interface AppStateProps {
  text: string;
}

interface AppDispatchProps {
  setText: (s: string) => void;
}

const App: React.FC<AppStateProps & AppDispatchProps> = props => {
  const { text, setText } = props;
  return (
    <div>
      value={text}
      <input
        type="text"
        value={text}
        onChange={event => setText(event.target.value)}
      />
    </div>
  );
};

export const DemoReduxReactRedux = () => {
  const RApp = connect(
    (initialState: State): AppStateProps => ({
      text: initialState.text
    }),
    (dispatch: Dispatch<Action>): AppDispatchProps => ({
      setText: (i: string) => dispatch({ type: SET_TEXT, payload: i })
    })
  )(App);
  return (
    <Provider store={store}>
      <RApp />
    </Provider>
  );
};


```

## ReactのuseRecuer

Reactの新しいAPIのHooksに導入されてuseReducerを使うやりかた。
子コンポーネントにコールバックを引き回す必要があるがdispatchだけを渡せばいいのでそこはuseStateに比べて簡素になった。
それでもdispatch引き回しがあるのは面倒だと思われる。

```tsx
const SET_TEXT = "SET_TEXT";

interface SetTextAction {
  type: typeof SET_TEXT;
  payload: string;
}

type Action = SetTextAction;

interface State {
  text: string;
}

const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case "SET_TEXT":
      return { text: action.payload };
    default:
      throw new Error("illegal action");
  }
};

const initialState: State = { text: "" };

export const DemoReduxUseReducer: React.FC = () => {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <div>
      value={state.text}
      <input
        type="text"
        value={state.text}
        onChange={event =>
          dispatch({ type: SET_TEXT, payload: event.target.value })
        }
      />
    </div>
  );
};

```



## React ReduxのHooks
react-reduxにもHooks APIを使う方法が導入された。

`useSelector`と`useDispatch`を使う。`useSelector`で状態を読み出し`useDispatch`で状態を書き換える。reducerなどは普通のreact-reduxと同じ。

`connect`でpropsのマッピングがする必要がなくなったのは楽になった思う。

```tsx
const SET_TEXT = "SET_TEXT";

interface SetTextAction {
  type: typeof SET_TEXT;
  payload: string;
}

type Action = SetTextAction;

interface State {
  text: string;
}

const reducer = (state: State = { text: "" }, action: Action): State => {
  switch (action.type) {
    case "SET_TEXT":
      return { text: action.payload };
    default:
      return state;
  }
};

const store = createStore<State, Action, {}, {}>(reducer);

const App: React.FC = () => {
  const text = useSelector((state: State) => state.text);
  const dispatch = useDispatch();
  return (
    <div>
      value={text}
      <input
        type="text"
        value={text}
        onChange={event =>
          dispatch({ type: SET_TEXT, payload: event.target.value })
        }
      />
    </div>
  );
};

export const DemoReduxReduxHooks = () => {
  return (
    <Provider store={store}>
      <App />
    </Provider>
  );
};

```

## まとめ

React ReduxのHooksを使うのが一番わかりやすいと思う。最初に一々マッピングを定義するのがボイラープレートぽくてめんどくさいし必要な状態が変わったときにコンポーネントの定義やマッピングの定義をいちいち書き換える手間が減ったのは楽になったように見える。
