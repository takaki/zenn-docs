---
emoji: "💻"
type: "tech"
published: true
title: JAX-RSでコンテキストの取得
topics: ["Java","JAX-RS"]
author: takaki@github
slide: false
---
JAX-RSのリソースクラスのメソッドの中で実行環境に関する情報(コンテキスト)を取得できる。何が取得できるかとはJSR-339のSection-9.2に記述してあるもの(Application・UriInfo・HttpHeaders・Requestなど)とサーブレットの場合は10.1に上げられているもの(ServletConfig・ServletContextなど)が取得できる。ただしJAX-RSを使っているときにサーブレットコンテキストに頼るような構成はあまりやらないほうが良いらしい。

具体的な情報は`@Context`アノテーションを使って取得する。以下サンプルとテスト。

```ContextResource.java
package example.jaxrs;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.core.Context;
import javax.ws.rs.core.HttpHeaders;
import javax.ws.rs.core.UriInfo;

@Path("/")
public class ContextResource {

    @GET
    @Path("uriinfo")
    public String uriinfo(@Context UriInfo uriInfo) {
        return uriInfo.getPath() + ";" + uriInfo.getQueryParameters().getFirst("foo");
    }

    @GET
    @Path("headers")
    public String headers(@Context HttpHeaders headers) {
        return headers.getHeaderString("X-Qiita-Demo");
    }
}
```

この例ではUriInfoを通してURIのパスとGETのパラメータを取得する例とHttpHeadersを通してHTTPのヘッダを取得する例を作成した。以下はテストコード。


```ContextResourceTest.groovy
package example.jaxrs

import org.glassfish.jersey.server.ResourceConfig
import org.glassfish.jersey.test.JerseyTest
import spock.lang.Shared
import spock.lang.Specification

import javax.ws.rs.core.Application

class ContextResourceTest extends Specification {
    @Shared
    def jerseyTest = new JerseyTest() {
        @Override
        protected Application configure() {
            return new ResourceConfig(ContextResource.class)
        }
    }

    def setupSpec() {
        jerseyTest.setUp()
    }

    def cleanupSpec() {
        jerseyTest.tearDown()
    }


    def "test uriinfo"() {
        when:
        def response = jerseyTest.target("/uriinfo").queryParam("foo", "bar").request().get()

        then:
        response.readEntity(String.class) == "uriinfo;bar"
    }

    def "test headers"() {
        when:
        def response = jerseyTest.target("/headers").request().header("X-Qiita-Demo", "Foo Bar").get()

        then:
        response.readEntity(String.class) == "Foo Bar"
    }
}
```

# 参考
* http://download.oracle.com/otndocs/jcp/jaxrs-2_0_rev_A-mrel-eval-spec/index.html - JSR-339(JRX-RS 2.0)

