---
title: Reactでinputを扱う(TypeScript版)
tags: React TypeScript
author: takaki@github
slide: false
---
* [Reeactでinputを扱う](https://qiita.com/takaki@github/items/8125535791b1bab8c85a)の改訂版

フォームの要素の`<input>`をReactで扱うと基本的にはReactがコントロールしてしまい編集できなくなる。

次のようにvalue属性を使うと何も編集できない。

```tsx
  const ValueOnly: React.FC = () => (
    <div>
      <label>nothing</label>
      <input type="text" value="0123" />
    </div>
  );
```

defaultValue属性を使うと編集できるがReact使ってinputの中身で何かやりたいのならほぼ無意味。

```tsx
  const DefaultValue: React.FC = () => (
    <div>
      <label>defaultValue</label>
      <input type="text" defaultValue="abcd" />
    </div>
  );
```

次が無難な例。
value属性をuseStateから作った値を使って表すことにすると中身も編集できて中身の値も利用できる。
onChangeにvalue属性の値を取得するハンドラを与えている。

```tsx
  const ValueWithOnChange: React.FC = () => {
    const [str, setStr] = useState("0123");
    return (
      <div>
        <label>value and handler: {str}</label>
        <input
          type="text"
          value={str}
          onChange={event => setStr(event.target.value)}
        />
      </div>
    );
  };
```

defaultValue属性を使った上でuseStateを併用してもいいがトリッキーなだけであまり意味は無さそう。
一応フォームの初期値は`abcd`になっている。

```tsx
  const DefaultValueWithOnChange: React.FC = () => {
    const [str, setStr] = useState("0123");
    return (
      <div>
        <label>defaultValue and handler: {str}</label>
        <input
          type="text"
          defaultValue="abcd"
          onChange={event => setStr(event.target.value)}
        />
      </div>
    );
  };
```

これは素直にvalue使えばという感じになる。

```tsx
  const DefaultValueWithOnChange2: React.FC = () => {
    const [str, setStr] = useState("0123");
    return (
      <div>
        <label>defaultValue and handler 2: {str}</label>
        <input
          type="text"
          defaultValue={str}
          onChange={event => setStr(event.target.defaultValue)}
        />
      </div>
    );
  };
```

