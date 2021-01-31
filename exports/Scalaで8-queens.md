---
title: Scalaで8-queens
tags: Scala
author: takaki@github
slide: false
---
Scalaを忘れていたのでリハビリ。Haskellで書いたものを移植してみたが末端が一個ずれることによるバグを直すのが面倒だった。

```scala
object EightQueens {

  def queen(n: Int): List[List[Int]] = n match {
    case 0 => List(List())
    case `n` => for {b <- queen(n - 1); q <- 0 to 7 if safe(q, b)} yield q :: b
  }

  def safe(q: Int, b: List[Int]) = b.indices.forall(i => checks(q, b, i))

  def checks(q: Int, b: List[Int], i: Int) = q != b(i) && (Math.abs(q - b(i)) != i + 1)

  def main(args: Array[String]) {
    val res = queen(8)
    res.foreach(l => println(l))
    println(res.length)
  }

}
```

