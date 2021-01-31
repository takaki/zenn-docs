---
emoji: "💻"
type: "tech"
published: true
title: ScalaFXを使ってみた
topics: ["Scala","JavaFX"]
author: takaki@github
slide: false
---
JavaのGUIライブラリのJavaFXは当然Scalaから使える。しかしScalaネイティブではないのでJavaFXをScalaらしく使えるようにしたライブラリがScalaFXである。次のJavaプログラムはボタンを表示してクリックするとコンソールに文字を出力する単純なプログラムである。


```java
import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.layout.VBox;
import javafx.stage.Stage;

public class FxButton extends Application {

    public static void main(final String[] args) {
        Application.launch(args);
    }

    @Override
    public void start(final Stage stage) throws Exception {
        Button button = new Button("Hello World");
        button.setOnMouseClicked(ev -> System.out.println("Hello World"));
        VBox vbox = new VBox(button);
        final Scene scene = new Scene(vbox);
        stage.setScene(scene);
        stage.setTitle("Hello");
        stage.show();
    }
}
```

これをScalaにそのまま書き直すと次のようになる。

```scala
import javafx.application.Application
import javafx.event.EventHandler
import javafx.scene.Scene
import javafx.scene.control.Button
import javafx.scene.input.MouseEvent
import javafx.scene.layout.VBox
import javafx.stage.Stage

object JavaFxHello {
  def main(args: Array[String]) {
    val hello = new JavaFxHello
    hello.run(args)
    // Application.launch(classOf[JavaFxHello], args: _*) // ここで呼ぶときはこちらを使う
    // Application.launch(args: _*) // NG
  }
}

class JavaFxHello extends Application {
  def run(args: Array[String]) {
    Application.launch(args: _*)
  }

  override def start(stage: Stage) {
    val button: Button = new Button("Hello World")

    val handler = new EventHandler[MouseEvent] {
      override def handle(t: MouseEvent) {
        println("Hello!")
      }
    }

    button.setOnMouseClicked(handler)
    val vbox: VBox = new VBox(button)
    val scene: Scene = new Scene(vbox)
    stage.setScene(scene)
    stage.setTitle("Hello")
    stage.show()

  }


}
```

わりとそのままである。Application.launchを呼ぶところが若干Javaと違った感じになっているのだがApplication.launchが中でちょっと変わったことをしているせいで`object`の中で呼ぶとき気をつけないといけない。あとイベントハンドラがJavaでlambdaで書けたものが後退している。これでも実際は問題ないと思うがScalaFXを使うともうちょっときれいに書ける。

```scala
package example.lambda

import scalafx.Includes._
import scalafx.application.JFXApp
import scalafx.application.JFXApp.PrimaryStage
import scalafx.scene.Scene
import scalafx.scene.control.Button
import scalafx.scene.layout.VBox

object ScalaFxHello extends JFXApp {
  stage = new PrimaryStage {
    title = "Hello"
    scene = new Scene {
      root = new VBox {
        children = new Button("Hello Button") {
          onMouseClicked = handle {
            println("hello")
          }
        }
      }
    }
  }
}
```

なおイベントを引数に取りたいイベントハンドラも次のようにScalaの関数でそのまま書けるようになる。

```scala
          onMouseReleased = (t:MouseEvent)=> println("Hello")
```

