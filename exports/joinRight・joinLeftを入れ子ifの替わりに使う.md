---
title: joinRight・joinLeftを入れ子ifの替わりに使う
tags: Scala
author: takaki@github
slide: false
---
`Either#joinRight`を使う。

入れ子になったEitherをフラットにするために使う。joinRightは`Either[A,Either[A,C]]`を`Either[A,C]`にする。
joinLeftは逆に`Either[Either[A,C],C]`を`Either[A,C]`にする。ifを使った複数条件のエラーチェックをするときに使える。

```scala
package example

object EitherDemo {
  def main(args: Array[String]) {
    println(oddAndPositive(-1))
    println(oddAndPositive(0))
    println(oddAndPositive(1))
    println(oddAndPositive2(-1))
    println(oddAndPositive2(0))
    println(oddAndPositive2(1))

    println(oddOrPositive(-2))
    println(oddOrPositive(-1))
    println(oddOrPositive(0))
    println(oddOrPositive2(-2))
    println(oddOrPositive2(-1))
    println(oddOrPositive2(0))

  }

  def mod(i: Int, j: Int) = (i % j + j) % j

  def oddOrPositive(i: Int): Either[String, Int] = {
    Either.cond(i >= 0, i, Either.cond(mod(i, 2) == 1, i, "even and negative number")).joinLeft
  }

  def oddOrPositive2(i: Int): Either[String, Int] = {
    if (i >= 0) {
      Right(i)
    } else if (mod(i, 2) == 1) {
      Right(i)a
    } else {
      Left("even and negative number")
    }
  }

  def oddAndPositive(i: Int): Either[String, Int] = {
    Either.cond(i >= 0, Either.cond(mod(i, 2) == 1, i, "even number"), "negative number").joinRight
  }

  def oddAndPositive2(i: Int): Either[String, Int] = {
    if (i >= 0) {
      if (mod(i, 2) == 1) {
        Right(i)
      } else {
        Left("even number")
      }
    } else {
      Left("negative number")
    }
  }
}
```
このようにifをネストさせるのではなくてEither.condをネストさせてからjoinRigth・joinLeftでネストを潰す。joinLeftがor条件・joinRightがand条件で正常系を出すと思えばよい。

