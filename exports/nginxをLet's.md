---
title: nginxをLet's encryptを使ってTLS/SSL化する
tags: nginx HTTP SSL証明書
author: takaki@github
slide: false
---
Let's encryptが使えるようになってからもかなり放置していたnginxのTLS/SSL化をやっと設定したのでメモ。

## 環境
* Debian 8.6(Jessie) + backports
* letsencrypt 0.9.3
* nginx 1.6.2

aptでインストールとかは省略するがLet's encryptのパッケージはbackportsから取ってくることに注意。

## 手順

certonlyで設定した。これは自分のnginxにLet's encryptのスクリプトが特殊なファイルを作成してそのファイルをLet's encrypt側で存在を確認してサーバーの実在を確認する…という流れになる。

通常のHTMLファイルなどをサービスしている`www.example.org`とアプリケーションサーバのプロキシーになっている`app.example.org`があるとするものとする。

具体的には`www.example.org`のための設定が以下のようだとする(例のためログの設定もしていないことに注意)。

```
server {
        listen 80;

        server_name www.example.org;

        root /var/www/html;
        index index.html;
}
```

次のコマンドを実行する。

```
# certbot certonly
```

ターミナルの対話環境で設定を入力していく。
最初の選択肢で `1 Place files in webroot directory`を選択する。ドメインに`www.example.org`を実行。webrootのディレクトリに`/var/www/html`を指定する。成功するとSSL証明書のファイルが`/etc/letsencrypt/live`の下に作成される。

次にこの証明書を使ってnginxを設定する。

```
server {
        listen 443;

        server_name app.example.org;

        root /var/www/html;
        index index.html;

        ssl on;
        ssl_certificate_key /etc/letsencrypt/live/www.example.org/privkey.pem;
        ssl_certificate /etc/letsencrypt/live/www.example.org/fullchain.pem;
}
```

これで`https://www.example.org`にアクセスしてSSLで通信ができるか確認する。なお，強制的にHTTPSにリダイレクトさせたい場合はHTTP側に次のように設定を追加する。なおcert.pemでは中間証明書が含まれないので使わない。

```
server {
        listen 80;
        listen [::]:80;

        server_name app.example.org;

        root /var/www/html;
        index index.html;
        return 301 https://$host$request_uri; # ここ
}
```

## 特殊な場合

アプリケーションサーバーを使っていて`/`以下が素直にアクセスできない場合以下のような回避策を使った。`.well-known`ディレクトリを使うのがわかっているので特別扱いした。

```
server {
        listen 80;
        listen [::]:80;

        server_name app.example.org;
        root /var/www/app;
        index index.html;

        # .well-known だけ通常のファイルシステムから見る
        location /.well-known/ {
                alias /var/www/app/.well-known/;
        }

        location / {
                # アプリサーバへの接続の設定などなど
        }
}
```

あとは`letsencrypt certonly`を実行しドメイン(`app.example.org`)とwebroot(`/var/www/app`)を指定する。HTTPSの設定やHTTPからのリダイレクトをする方法は前項と同様なので省略。

# メンテナンス
証明書の期限が3ヶ月なので定期的に更新する必要がある。次の通り。

```
# certbot renew
```

ちゃんと更新されるのか確認したいところではあるが古くなってないと更新されないので無理矢理更新するにはオプションをつけて実行する。

```
# certbot renew --force
```

これでコマンド出力で成功が確認できればOK。

# 参考
* http://qiita.com/kga/items/e30d668ec1ac5e30025b
* https://letsencrypt.jp/
* https://cryptoreport.websecurity.symantec.com/checker/ - 証明書の設定を確認してくれる
* https://www.ssllabs.com/ssltest/ - セキュリティ設定が適切かどうか確認してくれる

