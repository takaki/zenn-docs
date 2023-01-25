---
title: "Rust で GTK3 で GTK4 の違いを見る"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Rust, GTK]
published: true
---

https://gitlab.com/-/snippets/2488173

RustでGTK3を使ったプログラムをGTK4に移植する前に調査してみた。
サンプルプログラムを書いてどれぐらいの変化があるかということを確認した。

サンプルプログラムは以下の画像のようなもの。ボタンをクリックするとカウントが増える。

![デモ画面](/images/rust-gtk-3-and-4-demo.png)

crateは以下のバージョンを使っている。

```Cargo.toml
gtk = "0.16.2"
gtk4 = "0.5.5
```

まずはGTK3。

```rust
use std::sync::{Arc, Mutex};

use gtk::prelude::*;
use gtk::{Application, ApplicationWindow, Button, Entry, EntryBuffer, ListBox};

fn main() {
    let app = Application::builder()
        .application_id("org.gtk.example")
        .build();

    app.connect_activate(|app| {
        let window = ApplicationWindow::builder()
            .application(app)
            .title("First GTK+3 Program")
            .build();
        let list_box = ListBox::builder().build();

        let button = Button::with_label("click me");
        let count = Arc::new(Mutex::new(0));
        let entry_buffer = EntryBuffer::new(Some("count 0"));
        list_box.add(&button);
        let entry = Entry::builder()
            .editable(false)
            .buffer(&entry_buffer)
            .build();
        button.connect_clicked(move |_| {
            let mut c = count.lock().unwrap();
            *c += 1;
            entry_buffer.set_text(&format!("count {}", &c));
        });

        list_box.add(&entry);
        window.add(&list_box);
        window.show_all();
    });
    app.run();
}
```

次はGTK4。

```rust
use std::sync::{Arc, Mutex};

use gtk4::prelude::*;
use gtk4::{Application, ApplicationWindow, Button, Entry, EntryBuffer, ListBox};

fn main() {
    let app = Application::builder()
        .application_id("org.gtk.example")
        .build();

    app.connect_activate(|app| {
        let window = ApplicationWindow::builder()
            .application(app)
            .title("First GTK+4 Program")
            .build();
        let list_box = ListBox::builder().build();

        let button = Button::with_label("click me");
        let count = Arc::new(Mutex::new(0));
        let entry_buffer = EntryBuffer::new(Some("count 0"));
        list_box.append(&button);
        let entry = Entry::builder()
            .editable(false)
            .buffer(&entry_buffer)
            .build();
        button.connect_clicked(move |_| {
            let mut c = count.lock().unwrap();
            *c += 1;
            entry_buffer.set_text(&format!("count {}", &c));
        });

        list_box.append(&entry);
        window.set_child(Some(&list_box));
        window.show();
    });
    app.run();
}
```

プログラムの構造としてはほぼ同じだがコンテナに要素を追加するメソッドが`add`から`append`に変わっているなど微妙な差異が出ている。
当然コンパイルエラーになるので気付くのだがそれなりに面倒な感じがする。

