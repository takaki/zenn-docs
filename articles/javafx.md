---
emoji: "💻"
type: "tech"
published: true
title: JavaFXでスレッドを使って描画するときの注意
topics: ["Java","JavaFX"]
author: takaki@github
slide: false
---
JavaFX の描画をスレッドを使って行う。

* 環境: JDK8

例えば以下のようなコードを考える。

```java:ButtonLabelSleep.java
package example.javafx;

import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.Label;
import javafx.scene.layout.VBox;
import javafx.stage.Stage;

import java.time.LocalDateTime;

public class ButtonLabelSleep extends Application {
    public static void main(final String[] args) {
        launch(args);
    }

    @Override
    public void start(final Stage stage) throws Exception {

        final Label label = new Label("-");
        final Button button = new Button("Hello World");
        button.setOnMouseClicked(ev -> heavyMethod(label));
        final VBox vbox = new VBox(button, label);
        final Scene scene = new Scene(vbox);
        stage.setScene(scene);
        stage.setWidth(200);
        stage.setTitle(getClass().getSimpleName());
        stage.show();
    }

    private void heavyMethod(Label label) {
        try {
            label.setText("begin: " + LocalDateTime.now());
            Thread.sleep(3000);
            label.setText("end: " + LocalDateTime.now());
        } catch (final InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

}
```
ボタンをクリックしたらラベルに現在時刻を書き換えるという単純なプログラムだが実際ボタンを押すと描画が止まってしまう。これはJavaFX自体が自分のスレッドで動いているため `heavyMethod` 内で `Thread.sleep` で描画スレッドが止まってしまい描画が行われなくなってしまうからである。

ならば `haeavyMethod` を別のスレッドで動かせば固まらないかと思って書き直すと以下のようになる。

```java:ButtonSleepThreadFail.java
import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.Label;
import javafx.scene.layout.VBox;
import javafx.stage.Stage;

import java.time.LocalDateTime;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ButtonLabelSleepThreadFail extends Application {
    public static void main(final String[] args) {
        launch(args);
    }

    @Override
    public void start(final Stage stage) throws Exception {
        final ExecutorService service = Executors.newFixedThreadPool(5);

        final Label label = new Label("-");
        final Button button = new Button("Hello World");
        final Thread thread = new Thread(() -> heavyMethod(label));
        button.setOnMouseClicked(ev -> service.execute(thread));
        final VBox vbox = new VBox(button, label);
        final Scene scene = new Scene(vbox);
        stage.setScene(scene);
        stage.setWidth(200);
        stage.setTitle(getClass().getSimpleName());
        stage.show();
    }

    private void heavyMethod(Label label) {
        try {
            label.setText("begin: " + LocalDateTime.now());
            Thread.sleep(3000);
            label.setText("end: " + LocalDateTime.now());
        } catch (final InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

}
```
これで`heavyMethod`を別スレッドで動かすから大丈夫かと思いきや実際に動かすと以下のような例外が出てくる。

```
xception in thread "pool-2-thread-1" java.lang.IllegalStateException: Not on FX application thread; currentThread = pool-2-thread-1
	at com.sun.javafx.tk.Toolkit.checkFxUserThread(Toolkit.java:229)
	at com.sun.javafx.tk.quantum.QuantumToolkit.checkFxUserThread(QuantumToolkit.java:423)
	at javafx.scene.Parent$2.onProposedChange(Parent.java:367)
	at com.sun.javafx.collections.VetoableListDecorator.setAll(VetoableListDecorator.java:113)
	at com.sun.javafx.collections.VetoableListDecorator.setAll(VetoableListDecorator.java:108)
	at com.sun.javafx.scene.control.skin.LabeledSkinBase.updateChildren(LabeledSkinBase.java:575)
	at com.sun.javafx.scene.control.skin.LabeledSkinBase.handleControlPropertyChanged(LabeledSkinBase.java:204)
	at com.sun.javafx.scene.control.skin.LabelSkin.handleControlPropertyChanged(LabelSkin.java:49)
	at com.sun.javafx.scene.control.skin.BehaviorSkinBase.lambda$registerChangeListener$55(BehaviorSkinBase.java:197)
	at com.sun.javafx.scene.control.MultiplePropertyChangeListenerHandler$1.changed(MultiplePropertyChangeListenerHandler.java:55)
	at javafx.beans.value.WeakChangeListener.changed(WeakChangeListener.java:89)
	at com.sun.javafx.binding.ExpressionHelper$SingleChange.fireValueChangedEvent(ExpressionHelper.java:182)
	at com.sun.javafx.binding.ExpressionHelper.fireValueChangedEvent(ExpressionHelper.java:81)
	at javafx.beans.property.StringPropertyBase.fireValueChangedEvent(StringPropertyBase.java:103)
	at javafx.beans.property.StringPropertyBase.markInvalid(StringPropertyBase.java:110)
	at javafx.beans.property.StringPropertyBase.set(StringPropertyBase.java:144)
	at javafx.beans.property.StringPropertyBase.set(StringPropertyBase.java:49)
	at javafx.beans.property.StringProperty.setValue(StringProperty.java:65)
	at javafx.scene.control.Labeled.setText(Labeled.java:145)
	at example.javafx.ButtonLabelSleepThreadFail.heavyMethod(ButtonLabelSleepThreadFail.java:37)
	at example.javafx.ButtonLabelSleepThreadFail.lambda$start$0(ButtonLabelSleepThreadFail.java:25)
	at java.lang.Thread.run(Thread.java:745)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
```

JavaFXのウィジェットはJavaFXの自分のスレッド以外から触ろうとすることが禁じられているためである。
`Platform#runLator`の中で行うとJavaFXのスレッドの中で実行されるようになる。
ということで実際には以下のように少し書き換える。

```java:
package example.javafx;

import javafx.application.Application;
import javafx.application.Platform;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.Label;
import javafx.scene.layout.VBox;
import javafx.stage.Stage;

import java.time.LocalDateTime;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ButtonLabelSleepThreadPlatform extends Application {
    public static void main(final String[] args) {
        launch(args);
    }

    @Override
    public void start(final Stage stage) throws Exception {
        final ExecutorService service = Executors.newFixedThreadPool(5);

        final Label label = new Label("-");
        final Button button = new Button("Hello World");
        final Thread thread = new Thread(
                () -> Platform.runLater(() -> heavyMethod(label)));
        button.setOnMouseClicked(ev -> service.execute(thread));
        final VBox vbox = new VBox(button, label);
        final Scene scene = new Scene(vbox);
        stage.setScene(scene);
        stage.setWidth(200);
        stage.setTitle(getClass().getSimpleName());
        stage.show();
    }

    private void heavyMethod(Label label) {
        try {
            label.setText("begin: " + LocalDateTime.now());
            Thread.sleep(3000);
            label.setText("end: " + LocalDateTime.now());
        } catch (final InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

}
```

しかしこれはエラーは出なくなったもののボタンを押すと描画が停止するのは同じである。
これは`Platform#runLator` の中で`sleep`してしまっていて描画が止まるのは同じで書き換えた意味がない。

ということで個々の描画だけをJavaFXのスレッド内で行われるように改善しようとすると以下のようになる。

```java:ButtonLabelSleepThreadPlatform.java
package example.javafx;

import javafx.application.Application;
import javafx.application.Platform;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.Label;
import javafx.scene.layout.VBox;
import javafx.stage.Stage;

import java.time.LocalDateTime;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ButtonLabelSleepThreadPlatform extends Application {
    public static void main(final String[] args) {
        launch(args);
    }

    @Override
    public void start(final Stage stage) throws Exception {
        final ExecutorService service = Executors.newFixedThreadPool(5);

        final Label label = new Label("-");
        final Button button = new Button("Hello World");
        final Thread thread = new Thread(() -> heavyMethod(label));
        button.setOnMouseClicked(ev -> service.execute(thread));
        final VBox vbox = new VBox(button, label);
        final Scene scene = new Scene(vbox);
        stage.setScene(scene);
        stage.setWidth(200);
        stage.setTitle(getClass().getSimpleName());
        stage.show();
    }

    private void heavyMethod(Label label) {
        try {
            Platform.runLater(
                    () -> label.setText("begin: " + LocalDateTime.now()));
            Thread.sleep(3000);
            Platform.runLater(
                    () -> label.setText("end: " + LocalDateTime.now()));
        } catch (final InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

}
```

個々のウィジェットの書き換えを `Platform#runLator` の中で行うようにすれば良い。

