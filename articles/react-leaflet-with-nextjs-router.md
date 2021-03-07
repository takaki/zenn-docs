---
title: "React LeafletとNext.js Routerの組み合わせ "
emoji: "👌"
type: "tech"
topics: [react, reactrouter, leaflet, nestjs]
published: true
---
React LaefletというReactを使ってブラウザで地図を表示するライブラリがある。しかしReactなのでCSRであってSSRのNext.jsでは黒魔術を使わないと基本的に使えなかった。

しかしNext.js-10からはdynamic importを使うことで簡単に使えるようになった。

そのあたりの記述は以下の記事を参考。
https://stackoverflow.com/a/64634759/976917

これをもとにさらにNext.jsのrouterを使って地図を移動させたときブラウザのURLを書き換えて永続的なリンクを作成できるようにした。

なお使ったReact Leafletのバージョンは2.8.0である。React Leafletの3.0.0以降はライセンスがHippocratic Licenseというnon-OSS ライセンスになってしまったので使わない。
https://react-leaflet-v2-docs.netlify.app/en/

実際に作ったものは以下。

https://codesandbox.io/s/react-laeflet-nextjs-router-48gf8

通常のimportではなく `next/dynamic` を使ってdynamic importをしてReactのコンポーネントを読み込む。

```tsx
// index.tsx
const Map = useMemo(
    () =>
      dynamic(() => import("../components/map"), {
        loading: () => <p>A map is loading</p>,
        ssr: false
      }),
    []
  );
```

Laefletのコンポーネントが`component/map.tsx`になる。

Next.jsのrouterを読みこむ。

```tsx
const router = useRouter();
```
これはブラウザのHistory APIを使っている。`router.push`をすることでブラウザのURLを書き換えている。

https://nextjs.org/docs/api-reference/next/router

routerのqueryプロパティからパラメータを取得する。

```tsx
  const { lng, lat, zoom } = router.query;
```

パラメータの型が`string | string[]`なのでTypeScriptできちんと処理するのはちょっとだけ面倒。きちんと型を判定してから処理する。

```tsx
const parseParamInt = (p: string | string[]): number =>
  parseInt(Array.isArray(p) ? p[0] : p, 10);
```

あとはLeafletのdragendイベントとzoomendイベントにコールバックを用意する。
このコールバックで現在の中心点とズームレベルを取得してrouterを使ってURLを書き換えている。

```tsx
      <MapContainer
        center={[parseParamFloat(lat) || LAT, parseParamFloat(lng) || LNG]}
        zoom={parseParamInt(zoom) || ZOOM}
        scrollWheelZoom={true}
        style={{ height: 400, width: "100%" }}
        ondragend={(event) => {
          const z = event.target.getZoom();
          const c = event.target.getCenter();
          router.push(`?lng=${c.lng}&lat=${c.lat}&zoom=${z}`);
        }}
        onzoomend={(event) => {
          const z = event.target.getZoom();
          const c = event.target.getCenter();
          router.push(`?lng=${c.lng}&lat=${c.lat}&zoom=${z}`);
        }}
      >
```
