---
emoji: "💻"
type: "tech"
published: true
title: JAX-RSのBean Validationの基礎
topics: ["Java","JAX-RS","spock"]
---
JAX-RSにはBean Validationを使って渡されるパラメータを検証することができる。その使い方。

```build.gradle
    compile group: 'org.glassfish.jersey.ext', name: 'jersey-bean-validation', version: '2.23.1'
```

アノテーションを使ってパラメータを記述する。

```Hello.java
package example.jaxrs;

import javax.validation.constraints.NotNull;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.QueryParam;

@Path("/")
public class HelloValidateResource {
    @GET
    public String index(@NotNull @QueryParam("name") String name) {
        return "Hello " + name;
    }

}
```

これでGETでnameを受け取り，値無しは不許可ということになる。テストで確認する。

```HelloTest.groovy
package example.jaxrs

import org.glassfish.jersey.server.ResourceConfig
import org.glassfish.jersey.test.JerseyTest
import spock.lang.Shared
import spock.lang.Specification

import javax.ws.rs.core.Application

class HelloValidateResourceTest extends Specification {
    @Shared
    def jerseyTest = new JerseyTest() {
        @Override
        protected Application configure() {
            return new ResourceConfig(HelloValidateResource.class)
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
        def response = jerseyTest.target("/").queryParam("name", "World").request().get()

        then:
        response.getStatus() == 200
        response.readEntity(String.class) == "Hello World"
    }

    def "test fail"() {
        when:
        def response = jerseyTest.target("/").request().get()

        then:
        response.getStatus() != 200
    }

}
```
"test fail" にあるようにパラメータを省略するとHTTPリクエストが200にならない。
