* http://www.scala-graph.org/

Scalaでグラフ探索する必要があって自分でやるのは面倒でなんかあるだろと探したら発見した。

```build.sbt
libraryDependencies += "org.scala-graph" %% "graph-core" % "1.11.2"
```
として準備。


説明なしに単純な例。

```graphdemo.scala
package example.graph

import scalax.collection.Graph
import scalax.collection.GraphPredef._, scalax.collection.GraphEdge._

object GraphDemo1 {
  def main(args: Array[String]): Unit = {
    val g = Graph(1~2, 2~3, 4~5, 1~4, 4~3, 6~7)
    println(g.get(1).pathTo(g.get(3)))    //=> Some(Path(1, 1~4, 4, 4~3, 3))
    println(g.get(1).pathTo(g.get(2)))    //=> Some(Path(1, 1~2, 2))
    println(g.get(1).pathTo(g.get(6)))    //=> None

    val path = g.get(1).pathTo(g.get(3)).get
    println(path.size)   //=? 5
    println(path.weight) //=> 2
  }
}
```

重み付きとか有向グラフなども扱えるが当面必要ないので略。
