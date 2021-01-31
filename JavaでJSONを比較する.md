二つのJSONデータがあったときに比較をしたい。単に文字列で比較すると`[]`と`[ ]`ですら同じと判断できない。JSONを正規化する方法がどっかで決まっていたような記憶もあるがJavaのライブラリが探せなかった。

`org.json.json`を使ったらどうやら簡易的にできるようだ。

```build.gradle
dependencies {
    // https://mvnrepository.com/artifact/org.json/json
    compile group: 'org.json', name: 'json', version: '20160212'
}
```


```JsonCompare.java
package example.misc;

import org.json.JSONObject;

public class JsonCompare {

    public static void main(String[] args) {
        final JSONObject o1 = new JSONObject("{\'a\': 'hoge', b: 'bar', c: [1,3,5], bb: {e: 'あ'}}");
        final JSONObject o2 = new JSONObject("{b: 'bar',  \nc:[1,\n3,5], a: \n'hoge', \"bb\": {e: '\\u3042'}}");
        System.out.println(o1.toString());
        System.out.println(o2.toString());
        System.out.println(o1.toString().equals(o2.toString()));
    }
}
```

出力は次のとおり。

```
{"bb":{"e":"あ"},"a":"hoge","b":"bar","c":[1,3,5]}
{"bb":{"e":"あ"},"a":"hoge","b":"bar","c":[1,3,5]}
true
```
キーの並び順序が気にはなるがこれでJSONの比較ができそう？複雑なJSONでも大丈夫なのかは未確認。

# 参考
* https://github.com/skyscreamer/JSONassert - ユニットテストでいろいろやるならこれがいいのかも？
* https://github.com/lukas-krecan/JsonUnit - こういうのもある

ちゃんとやるならこういうのを使ったほうがいいのだろう。
