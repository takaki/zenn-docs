---
title: "Let's encryptでワイルドカード証明書を作る"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SSL証明書"]
published: true
---
Let's encryptでワイルドカード証明書を作成した。

最初にLet's encryptを導入したときにはワイルドカード証明書に対応してなかったので導入してなかった。今さらだが取得して順次適用予定。

取得するにはDNSの設定変更ができることが必要。
一つのFQDNだとwebサーバ側の変更で所持していることを確認していたがワイルドカードなのでDNSのリソースレコードの編集ができることを確認して発行される。

example.orgの証明書を発行するものとする。

```
# certbot certonly --manual --preferred-challenges dns-01 -m me@example.org -d '*.example.org' -d example.org
```

`--preferred-challenges dns-01` でDNSを使った認証を指定している。

`--manual` をつけないとwebベースの認証になってしまうので必要。

```
debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Requesting a certificate for *.example.org and example.org
Performing the following challenges:
dns-01 challenge for example.org
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.example.org with the following value:
cjCnEhj1ko3PP2Gy3fxT9cN45-FCLjTPH94VpmnVI9w
Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

こんな感じで一旦止まる。

ここで表示された通りトークンをDNSの`_acme-challenge.example.org`のTXTレコードとして追加する。

その後Enterを入力すると以下のように表示されて証明書が保存される。あとは今までの証明書と置き換えて使う。

```
Waiting for verification...
Cleaning up challenges
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/example.org/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/example.org/privkey.pem
   Your certificate will expire on 2022-07-04. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again. To non-interactively renew *all* of your
   certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:
   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

