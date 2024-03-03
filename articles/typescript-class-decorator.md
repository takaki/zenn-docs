---
title: "TypeScriptのclass decoratorを試す"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["TypeScript"]
published: false
---
TypeScriptであるベースクラスのサブクラス一覧を保持しておいて文字列からサブクラスインスタンスを作るようなことをしたかった。
Rubyだと組込でそんなことができたりするがTypeScript(JavaScript)だとそんなものはなくてなんとかならないかと試行錯誤していた。
クラスのstatic blockを使って定型文を書くという手法もあるがもうちょっとわかりやすい方法はないかと考えてdecoratorでなんとかなりそうと使ってみた。
まだ正式にリリースされてないので参考資料も少なくて苦労した。この文法はstage3を元にしている。stage2の資料もみつかるが互換性はないので注意。

なおサンプルはswc-nodeとdenoで動作を確認した。
ts-nodeだと`@`が文法違反という認識をされてしまった。何かの設定で解決するのか調べてみたがわからなかった。

```ts
type BaseClassDecorator = (
  value: new () => BaseClass, // Functionより狭くすることでBaseClassのサブクラスのみにできる
  context: ClassDecoratorContext,
) => void | (new () => any);

function createInjections() {
  const injections: Record<string, new () => BaseClass> = {};

  function inject(injectionKey: string): BaseClassDecorator {
    return (v, context) => {
      // console.log({ v, context });
      injections[injectionKey] = v;
    };
  }

  function getInjections() {
    return injections;
  }

  return { inject, getInjections };
}

const { inject, getInjections } = createInjections();

abstract class BaseClass {}

// これはtypescriptの文法違反になる
// @inject('not subclass')
// class NotBase {}

@inject('test1')
class Test1 extends BaseClass {}

const injections = getInjections();
// console.log({ injections });

console.log(new injections['test1']());
```

## 参考文献

https://github.com/tc39/proposal-decorators

https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/#decorators

公式資料以外はあまりない。
