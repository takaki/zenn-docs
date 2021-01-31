---
emoji: "💻"
type: "tech"
published: true
title: JAX-RSのBean Validationの例
topics: ["Java","JAX-RS","spock"]
author: takaki@github
slide: false
---
JAX-RSのBean Validationのいくつかの例を上げる。

validatorは`javax.validation.constraints`にある。例えば次のようになる。
`@DecimalMax`と`@DecimalMin`は名前の通り整数値の下限・上限を定める。`@Pattern`は正規表現で文字列のパターンを定める。

```java

import javax.validation.constraints.DecimalMax;
import javax.validation.constraints.DecimalMin;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Pattern;

// ...
// ......
// ...

@Path("/")
public class HelloValidateSampleResource {
    @GET
    @Path("minmax")
    public String minmax(
            @NotNull @DecimalMax(value = "10") @DecimalMin(value = "3") @QueryParam("number") int number) {
        return Integer.toString(number * 10);
    }

    @GET
    @Path("pattern")
    public String pattern(
            @NotNull @Pattern(regexp = "a+b*c") @QueryParam("pattern") String pattern) {
        return "pattern = " + pattern;
    }
}
```

実際にテストしてみる。

```groovy
class HelloValidateSampleResourceTest extends Specification {
    @Shared
    def jerseyTest = new JerseyTest() {
        @Override
        protected Application configure() {
            return new ResourceConfig(HelloValidateSampleResource.class).
                    property(ServerProperties.BV_DISABLE_VALIDATE_ON_EXECUTABLE_OVERRIDE_CHECK, true)
        }
    }

    def setupSpec() {
        jerseyTest.setUp()
    }

    def cleanupSpec() {
        jerseyTest.tearDown()
    }

    def "test minmax"() {
        when:
        def response = jerseyTest.target("/minmax").queryParam("number", "5").request().get()

        then:
        response.getStatus() == 200
        response.readEntity(String.class) == "50"

        expect:
        jerseyTest.target("/minmax").request().get().getStatus() != 200
        jerseyTest.target("/minmax").queryParam("number", "20").request().get().getStatus() != 200
        jerseyTest.target("/minmax").queryParam("number", "1").request().get().getStatus() != 200
    }

    def "test pattern" () {
        expect:
        jerseyTest.target("/pattern").queryParam("pattern", "abc").request().get().getStatus() == 200
        jerseyTest.target("/pattern").queryParam("pattern", "aac").request().get().getStatus() == 200

        jerseyTest.target("/pattern").queryParam("pattern", "xxx").request().get().getStatus() != 200
        jerseyTest.target("/pattern").queryParam("pattern", "xabc").request().get().getStatus() != 200
    }
}
```

また`org.hibernate.validator.constraints`にもvalidatorが定義されている。その例。

```java
import org.hibernate.validator.constraints.Email;
import org.hibernate.validator.constraints.Length;

// ...
// ......
// ...

    @GET
    @Path("length")
    public String length(@NotNull @Length(min = 3, max = 10) @QueryParam("str") String str) {
        return str;
    }

    @GET
    @Path("email")
    public String email(@NotNull @Email @QueryParam("email") String email) {
        return email;
    }
}

```

テストは次の通り。

```groovy
    def "test length" () {
        expect:
        jerseyTest.target("/length").queryParam("str", "1234567").request().get().getStatus() == 200
        jerseyTest.target("/length").queryParam("str", "123").request().get().getStatus() == 200

        jerseyTest.target("/length").queryParam("str", "abcdefghijk").request().get().getStatus() != 200
        jerseyTest.target("/length").queryParam("str", "aa").request().get().getStatus() != 200
    }

    def "test email" () {
        expect:
        jerseyTest.target("/email").queryParam("email", "test@example.org").request().get().getStatus() == 200
        jerseyTest.target("/email").queryParam("email", "test@localhost").request().get().getStatus() == 200

        jerseyTest.target("/email").queryParam("email", "test").request().get().getStatus() != 200
    }
```

* http://qiita.com/takaki@github/items/40f8bc02dc06a0e2848b
