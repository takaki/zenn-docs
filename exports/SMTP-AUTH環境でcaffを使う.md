---
title: SMTP-AUTH環境でcaffを使う
tags: GnuPG Perl SMTP
author: takaki@github
slide: false
---
GPGの鍵署名手続を大幅に楽にしてくれるツールにcaffがある。久しぶりにDebian関係でキーサインパーティーに参加してさてさてcaffをとやったらSMTPサーバ環境がSMTP-AUTHが必須になってメールが送付できなくなっていた。caffの設定をどうすりゃいいのだと調べたが要するにPerlのMail::Mailerの設定を`.caffrc`にするということだった。具体的に以下の通り。

```pl:.caffrc
$CONFIG{'mailer-send'} = [ 'smtp', Server => 'smtp.example.org',
                                    Port=> 587,
                                    From => $CONFIG{'email'},
                                    Auth => ['AUTHUSER', 'AUTHPASS' ],
                                    Debug =>  0 ];
```

とかやっとけば自分の使っているメールサーバではOKだった。`smtp`を`smtps`にするとか`Port`を変更するとか環境によって適当に変更しとけばなんとかなるだろう。

