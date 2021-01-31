---
emoji: "💻"
type: "tech"
published: true
title: JavaとGroovyの整数リテラルの違い
topics: ["Java","Groovy","JUnit","spock"]
author: takaki@github
slide: false
---
Javaで書いたユニットテストをSpock(Groovy)に書き直して整数リテラルの違いでテストが失敗したのでメモ。

```java:test.java
package example

import org.junit.Test;

import static org.hamcrest.core.Is.is;
import static org.junit.Assert.assertThat;

public class IntTest {
    @Test
    public void test() {
        assertThat(-1, is(0xffffffff));
    }
}
```

をSpockのテストにそのまま書き直すと

```Groovy:test.groovy
package example

import spock.lang.Specification

class IntSpec extends Specification {
    def "test"() {
        expect:
        -1 == 0xffffffff
    }
}
```

となるがこれは

```
Condition not satisfied:

-1 == 0xffffffff
   |
   false

Expected :4294967295

Actual   :-1
```

ということになってしまうのでちょっと書き直す。

```groovy:test.groovy
package example

import spock.lang.Specification

class IntSpec extends Specification {
    def "test"() {
        expect:
        -1 == (int) 0xffffffff
    }

}
```
(int)でキャストするのがポイント。

