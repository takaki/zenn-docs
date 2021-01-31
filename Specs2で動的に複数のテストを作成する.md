すでにファイルにテスト対象のデータがあってそれを読み取りながらScalaのSpecs2でテストをしたいと考えた。ファイルに書かれている各々のテストケースをSpecs2の一つのテストとして扱いたいとする。動的にテストを生成した上で実行にFragment#foreachを使う。次のようになる。

```scala
import org.junit.runner.RunWith
import org.specs2.mutable.Specification
import org.specs2.runner.JUnitRunner
import org.specs2.specification.core.Fragment

class ParamSpecs extends Specification {
  "paramtest" >> {
    Fragment.foreach(Seq("1", "2", "0", "3"))(e => s"param test $e" >> {
      e.toInt must be_>=(1)
    })
  }
}
```

実行すると

```
Testing started at 10:42 ...

0 is less than 1
java.lang.Exception: 0 is less than 1
	at org.specs2.matcher.MatchResultStackTrace$class.setStacktrace(Expectations.scala:57)
	at org.specs2.mutable.Specification.setStacktrace(Specification.scala:15)
	at org.specs2.matcher.ExpectationsCreation$class.checkFailure(Expectations.scala:37)
	at org.specs2.mutable.Specification.checkFailure(Specification.scala:15)
...中略...
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)

paramtest
param test 1
param test 2
param test 3
```
こんな感じの出力になる。
