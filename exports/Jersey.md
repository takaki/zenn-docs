---
title: Jersey Test FrameworkをSpockで動かす
tags: Java JAX-RS spock
author: takaki@github
slide: false
---
JAX-RSのユニットテストのライブラリにJersey Test Frameworkがある。ドキュメントはJUnitの説明だけなのでSpockで動かす例を説明する。

```build.gradle
Dependencies {
    compile group: 'javax.ws.rs', name: 'javax.ws.rs-api', version: '2.0.1'

    testCompile group: 'org.glassfish.jersey.test-framework.providers', name: 'jersey-test-framework-provider-grizzly2', version: '2.23.1'
    testCompile group: 'org.spockframework', name: 'spock-core', version: '1.0-groovy-2.4'

}
```

JAX-RSとJersey Test Frameworkのライブラリを追加。実はJersey Testには目的によっていくつかのテストを動かすコンテナが用意されいるが詳しくはドキュメント参考。

サンプルは次

```java:HelloResource.java
package example.jaxrs;

import javax.ws.rs.GET;
import javax.ws.rs.Path;

@Path("/")
public class HelloResource {
    @GET
    public String index() {
        return "Hello World";
    }
}
```

次にSpockのテストケース。ポイントはJerseyTestの扱い。JUniteではJerseyTestクラスを継承しているがSpockでは多重継承になるのでできない。そこでJerseyTestのインスタンスを作って実行することになる。

```HelloValidateResource.groovy
package example.jaxrs

import org.glassfish.jersey.server.ResourceConfig
import org.glassfish.jersey.test.JerseyTest
import spock.lang.Shared
import spock.lang.Specification

import javax.ws.rs.core.Application
import javax.ws.rs.core.Response

class HelloResourceTest extends Specification {
    @Shared
    def jerseyTest = new JerseyTest() {
        @Override
        protected Application configure() {
            return new ResourceConfig(HelloResource.class)
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

    def "test index 2"() {
        when:
        def response = jerseyTest.target("/").request().get()

        then:
        response.getStatus() == 200
        response.readEntity(String.class) == "Hello World"
    }

}
```
このようになる。

# 参考
* https://jersey.java.net/documentation/latest/user-guide.html#test-framework

