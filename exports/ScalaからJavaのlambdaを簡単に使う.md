---
title: ScalaからJavaのlambdaを簡単に使う
tags: Scala
author: takaki@github
slide: false
---
ScalaからJavaのライブラリを使っていてJavaのlambdaを使いたいときかなり面倒なことになる。例としてJavaのOptional#mapをScalaから使うことを考える。

```scala
import java.util.Optional

object JavaLambda {
  def main(args: Array[String]) {
    val s = Optional.of("Scala")


    val l = s.map((s:String)=>s.concat("!!!")).get() // Error!!

    println(l)

  }
}
```

まずこれはコンパイルが通らない。Optional#mapがScalaの関数を受け付けない。動くようにするには`java.util.function`を使ってJavaの無名クラスを作ることになる。

```scala
import java.util.Optional
import java.util.function.Function

object JavaLambda {
  def main(args: Array[String]) {
    val s = Optional.of("Scala")

    val mapper = new Function[String, String] {
      override def apply(v1: String):String = v1.concat("!!!")
    }

    val l = s.map(mapper).get()

    println(l)

  }
}
```

しかし，これはみっともない。これを解決するのにscala-java8-compatというライブラリがある。これを使うと次のように書ける。

```scala
import java.util.Optional
import scala.compat.java8.FunctionConverters._

object Java8Lambda {
  def main(args: Array[String]) {
    val s = Optional.of("Scala")

    val l = s.map(((v: String) => v.concat("!!!")).asJava).get()

    println(l)

  }
}
```
asJavaがJavaの無名関数クラスに適切に変換してくれる。

`@FunctionalInterface`にも適当に変換してくれるともっと楽なんだが。

* https://github.com/scala/scala-java8-compat

