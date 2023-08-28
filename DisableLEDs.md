# ゼロくらいから始めるBOINC(LEDの消灯制御)

このテキストは「ラズパイって聞いたことはあるけどよく知らん」とか
「レトロパイ？なら入れたことがある」くらいの人が、
「ディスプレイとキーボード無しでOSインストール」して「BOINCをインストールする」ところまでを
メモしたものです

[TOC]

## ・OS起動時に実行するスクリプトの作成

rc.localファイルを作成

```
$ sudo nano /etc/rc.local
```

初期状態では「rc.local」は無効化されている(ファイルが無い)ので新規に作成する。
記述内容は以下の通り

    #!/bin/sh
    echo none | sudo tee /sys/class/leds/led0/trigger
    echo none | sudo tee /sys/class/leds/led1/trigger
    echo 0 | sudo tee /sys/class/leds/led0/brightness
    echo 0 | sudo tee /sys/class/leds/led1/brightness
> 「led0」と「led1」が無い場合、それぞれ「ACT」と「PWR」に読み替える

## ・systemdで起動させる設定をする

まず実行権限を付与

```
$ sudo chmod u+x /etc/rc.local
```
サービスファイルをコピー

```
$ sudo cp /lib/systemd/system/rc-local.service /etc/systemd/system/.
```

サービスファイルに対してリンクを貼る

```
$ sudo ln -s /etc/systemd/system/rc-local.service /etc/systemd/system/multi-user.target.wants
```

再起動

```
$ sudo reboot
```
再起動後、OSの起動完了時から基板上のLEDが両方とも消灯すれば成功
OSのシャットダウン時に点灯するので再起動時の目安になる
