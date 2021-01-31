---
emoji: "💻"
type: "tech"
published: true
title: HMACをいろんな言語で
topics: ["Java","Python","Ruby"]
author: takaki@github
slide: false
---
HMAC SHA-256 のサンプル。暇があったら他も書きます。

秘密鍵 `secret key`・対象 `This is a pen.` で HMAC SHA-256 でサイン。
出力は`b4197908ed255480db6bbedb27426c89520bcd3b79c81006f70a94f415b43a43`になります。

まずJava。

```java
package example.misc;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;

public class HmacDemo {

    private static final String SECRET = "secret key";
    private static final String TEXT = "This is a pen.";

    public static void main(String[] args) throws Exception {
        String algo = "HMacSHA256";
        final SecretKeySpec keySpec = new SecretKeySpec(SECRET.getBytes(),
                algo);
        final Mac mac = Mac.getInstance(algo);
        mac.init(keySpec);
        final byte[] signBytes = mac.doFinal(TEXT.getBytes());
        for (byte signByte : signBytes) {
            System.out.printf("%02x", signByte & 0xff);
        }
        System.out.print("\n");

    }
}
```
アルゴリズムの指定が冗長なんだけどこういう書き方しかないのか。byte列の16進出力もうちょっときれいにできるとうれしいのだが。Base64エンコードのライブラリはbyte列をそのまま受け付けるこの場合はいいのかもしれない。

次はPython

```python3
#!/usr/bin/python3

import hashlib
import hmac

SECRET = "secret key"
TEXT = "This is a pen."

print(hmac.new(bytes(SECRET, 'ascii'), bytes(TEXT, 'ascii'), hashlib.sha256).hexdigest())
```


次Ruby。

```ruby
#!/usr/bin/ruby

require 'openssl'

SECRET = "secret key"
TEXT = "This is a pen."

puts OpenSSL::HMAC.digest(OpenSSL::Digest::Digest.new('sha256'),
  SECRET, TEXT).unpack("H*")
```

