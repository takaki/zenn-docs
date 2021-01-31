---
title: Gsonを使ってJSONのpretty print
topics: ["Java","JSON","GSON"]
type: "tech"
emoji: "💻"
published: true
---

Gsonを使ってJSONのpretty print。ワンライナー風。

```java:GsonJsonPP.java
package org.example.javalabo;

import com.google.gson.GsonBuilder;
import com.google.gson.JsonParser;

public class GsonJsonPP {
    public static void main(String[] args) {
        System.out.println(new GsonBuilder().setPrettyPrinting().create()
                .toJson(new JsonParser()
                        .parse("{\"aaa\":0, \"bbb\": [\"ddd\", \"eee\"], \"fff\":{\"ggg\":\"hhh\"}}")));
    }
}
```

こうなる。

```js:出力
{
  "aaa": 0,
  "bbb": [
    "ddd",
    "eee"
  ],
  "fff": {
    "ggg": "hhh"
  }
}
```
