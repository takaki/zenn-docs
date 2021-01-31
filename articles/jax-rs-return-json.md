---
emoji: "💻"
type: "tech"
published: true
title: JAX-RSでJSONを返す
topics: ["Java","JAXRS","JSON"]
author: takaki@github
slide: false
---
JAX-RSでJSONを返すのは簡単。JSONをサポートする拡張モジュールを導入してアノテーションで返却するメディアタイプを`application/json`を指定すれば良い。拡張モジュールがJSONに変換できるオブジェクトだったら面倒見てくれる。

```build.gradle
dependencies {
    compile group: 'org.glassfish.jersey.media', name: 'jersey-media-json-jackson', version: '2.23.1'

}
```
JerseyにはいくつかのJSON変換モジュールが用意されているがこれはJacksonを使ったモジュール。


リソースクラスのメソッドに`@Produces`を使ってメディアタイプを指定する。


```JsonResource.java
package example.jaxrs;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Path("/")
public class JsonResource {
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public String index() {
        return "Hello World";
    }

    @GET
    @Path("list")
    @Produces(MediaType.APPLICATION_JSON)
    public List<String> list() {
        return Arrays.asList("a","b");
    }

    @GET
    @Path("map")
    @Produces(MediaType.APPLICATION_JSON)
    public Map<String, String> map() {
        Map<String, String> map = new HashMap<>();
        map.put("a", "000");
        map.put("b", "111");
        return map;
    }

    static public class Person{
        public String name;
        public int age;
    };
    @GET
    @Path("pojo")
    @Produces(MediaType.APPLICATION_JSON)
    public Person pojo() {
        Person person = new Person();
        person.name = "Foo Bar";
        person.age = 30;
        return person;
    }
}
```

プリミティブ・リスト・マップ・POJOなら適当に面倒見てくれる。テストコードを見てどのように変換されるか確認。JSONの確認にはGroovy側のJsonSlurperを使った。

```JsonResourceTest.groovy
package example.jaxrs

import groovy.json.JsonSlurper
import org.glassfish.jersey.server.ResourceConfig
import org.glassfish.jersey.test.JerseyTest
import spock.lang.Shared
import spock.lang.Specification

import javax.ws.rs.core.Application

class JsonResourceTest extends Specification {

    def jsonslurper = new JsonSlurper()

    @Shared
    def jerseyTest = new JerseyTest() {
        @Override
        protected Application configure() {
            return new ResourceConfig(JsonResource.class)
        }
    }

    def setupSpec() {
        jerseyTest.setUp()
    }

    def cleanupSpec() {
        jerseyTest.tearDown()
    }

    def "test index"() {
        when:
        def response = jerseyTest.target("/").request().get(String.class)
        then:
        response == "Hello World"
    }

    def "test list"() {
        when:
        def response = jerseyTest.target("/list").request().get(String.class)
        then:
        jsonslurper.parseText(response) == jsonslurper.parseText('''["a", "b"]''')

    }

    def "test map"() {
        when:
        def response = jerseyTest.target("/map").request().get(String.class)
        then:
        jsonslurper.parseText(response) == jsonslurper.parseText('''{"b": "111", "a": "000" }''')
    }

    def "test pojo"() {
        when:
        def response = jerseyTest.target("/pojo").request().get(String.class)
        then:
        jsonslurper.parseText(response) == jsonslurper.parseText('''{"name": "Foo Bar", "age": 30 }''')
    }
}
```

