---
title: Linuxでbootable USB diskイメージを作成する
tags: USB
author: takaki@github
slide: false
---
ISO9660形式で配布されているインストーラをCDやDVDに焼かずにUSBメモリからブートするためのイメージを作成した。

ダウンロードしたISOイメージをisohybrid で変換する。Debianではsyslinux-utilsパッケージに含まれている。
```% isohybrid installer.iso```
そのままこのファイルがUSBメモリでブート可能な形式に書き換えられる。次に dd を使って書き込む
```% sudo dd if=installer.iso of=/dev/sdX```
`/dev/sdX`は適当なデバイス名(/dev/sdfとか/dev/sdgとか)を指定する。何になるかはデバイスを接続したときに
`dmesg`もしくは`syslog`の出力を見る。またGNOMEやKDEだと自動的にマウントしてしまうことがあるのでその場合は手動でデバイスを事前に`umount`する

ddの書き込みが終了すればOK。SystemRescueCDやUbuntuで動作確認あり。


