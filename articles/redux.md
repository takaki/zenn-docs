---
emoji: "💻"
type: "tech"
published: true
title: redux で非同期アクションを使う
topics: ["TypeScript","React","redux"]
author: takaki@github
slide: false
---
redux-promiseの使い方 (https://qiita.com/takaki@github/items/42bddf01d36dc18bdc8e) の改訂版

reduxに対して非同期アクションは使えるのか調べたら素直に使えるようになっていた。以前はredux-promiseというミドルウェアを使っていたが必要なくなっていた。

```tsx
import React from "react";
import { connect, Provider } from "react-redux";
import { createStore, Dispatch } from "redux";
import axios from "axios";

interface StateProps {
  time: string;
}

interface DispatchProps {
  setTime: () => void;
  reset: () => void;
}

const App: React.FC<StateProps & DispatchProps> = props => (
  <div>
    <h1>time = {props.time}</h1>
    <button onClick={props.setTime}>Get</button>
    <button onClick={props.reset}>Reset</button>
  </div>
);

const RESET = "RESET";
const SET_TIME = "SET_TIME";

interface Reset {
  type: typeof RESET;
}

interface SetTime {
  type: typeof SET_TIME;
  payload: string;
}

type Action = Reset | SetTime;

interface State {
  time: string;
}

const reducer = (state: State = { time: "" }, action: Action):State => {
  switch (action.type) {
    case "RESET":
      return { time: "" };
    case "SET_TIME":
      return { time: action.payload };
    default:
      return state;
  }
};

const mapStateToProps = (state: State): StateProps => ({
  time: state.time
});

// Promiseを返すアクション
const setTime = async (): Promise<SetTime> => {
  const response = await axios.get("https://ntp-a1.nict.go.jp/cgi-bin/json");
  const time = response.data.st;
  return {
    type: SET_TIME,
    payload: time
  };
};

const reset = (): Reset => ({
  type: RESET
});

// このようにdispatchを呼ぶときにawait asyncをつけるだけでPromiseを返すactionが普通に使えるようになる
const mapDispatchToProps = (dispatch: Dispatch<Action>): DispatchProps => ({
  setTime: async () => dispatch(await setTime()),
  reset: () => dispatch(reset())
});

const store = createStore<State, Action, {}, {}>(reducer, { time: "" });
const RApp = connect(mapStateToProps, mapDispatchToProps)(App);

export const DemoReduxPromise = () =>  (
  <Provider store={store}>
    <RApp />
  </Provider>
);
```

## Redux Hooks版
```tsx
import axios from "axios";
import * as React from "react";
import { Provider, useDispatch, useSelector } from "react-redux";
import { createStore } from "redux";

interface State {
  time: string;
}

// Promiseを返すアクション
const doSetTime = async (): Promise<SetTime> => {
  const response = await axios.get("https://ntp-a1.nict.go.jp/cgi-bin/json");
  const time = response.data.st;
  return {
    type: SET_TIME,
    payload: time
  };
};

const doReset = (): Reset => ({
  type: RESET
});

const App: React.FC = () => {
  const time = useSelector((s: State) => s.time);
  const dispatch = useDispatch();
  // そのままで扱うとランタイムでエラーになる
  // const setTime = () => dispatch(doSetTime());
  // このようにdispatchを呼ぶときにawaitとasyncをつけるだけでPromiseを返すactionが普通に使えるようになる
  const setTime = async () => dispatch(await doSetTime());
  const reset = () => dispatch(doReset());
  return (
    <div>
      <h1>time = {time}</h1>
      <button onClick={setTime}>Get</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
};

const RESET = "RESET";
const SET_TIME = "SET_TIME";

interface Reset {
  type: typeof RESET;
}

interface SetTime {
  type: typeof SET_TIME;
  payload: string;
}

type Action = Reset | SetTime;

const reducer = (state: State = { time: "" }, action: Action): State => {
  switch (action.type) {
    case "RESET":
      return { time: "" };
    case "SET_TIME":
      return { time: action.payload };
    default:
      return state;
  }
};

const store = createStore(reducer);

export const DemoReduxPromise = () => (
  <Provider store={store}>
    <App />
  </Provider>
);
```

