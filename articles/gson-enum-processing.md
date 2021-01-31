---
emoji: "💻"
type: "tech"
published: true
title: GsonでEnumを処理する
topics: ["GSON","Java","JSON"]
author: takaki@github
slide: false
---
* 環境：JDK8, gson-2.5

[Gsonでenumをいい感じに処理する](http://qiita.com/kazy/items/3fd0aa15345b374e7b6c) を見て気になったので。

```java:GsonEnum.java
package org.example.javalabo;

import com.google.gson.Gson;
import com.google.gson.annotations.SerializedName;

import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.core.Is.is;

public class GsonEnum {
    public static void main(String args[]) {
        Gson gson = new Gson();
        assertThat(gson.fromJson("0", Platform.class), is(Platform.Twitter));
        assertThat(gson.fromJson("1", Platform.class), is(Platform.TwitterDM));
        assertThat(gson.toJson(Platform.Twitter), is("\"0\""));
        assertThat(gson.toJson(Platform.TwitterDM), is("\"1\""));
    }

    public enum Platform {
        @SerializedName("0")
        Twitter,
        @SerializedName("1")
        TwitterDM;
    }

}
```
`int`と`String`で変換がちょっとずれるということに目を潰れば`@SerializedName`で簡単。
JSON と Enum の対応が String で良いなら何も問題はないと思われます。

