---
emoji: "💻"
type: "tech"
published: true
title: iptablesをnftablesに変換してみた
topics: ["Linux","iptables","nftables"]
author: takaki@github
slide: false
---
iptablesで作ったパケットフィルタリングルールをnftablesに変換してみた。
以下はよくある入力は特定ポートしか受け付けなくするiptablesのスクリプト

```bash
#!/bin/sh
iptables -F
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT

iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT

iptables -A INPUT -p tcp -m tcp --dport ssh -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport smtp -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport domain -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport http -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport https -j ACCEPT
iptables -A INPUT -p udp -m udp --dport domain -j ACCEPT

iptables -I INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
```

これをnftablesでやると以下にようになる。

```bash
#!/usr/sbin/nft -f
flush table filter

table filter {
        chain input {
                type filter hook input priority 0; policy accept;
                ct state established accept
                ct state related accept
                iif lo accept
                tcp dport {ssh,smtp,domain,http,https} accept
                udp dport domain accept
                ip protocol icmp accept
                counter drop
        }

        chain output {
                type filter hook output priority 0; policy accept;
        }
}
```

