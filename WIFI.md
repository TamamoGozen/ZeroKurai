# ゼロくらいから始めるBOINC(Wi-Fi設定)

このテキストは「ラズパイって聞いたことはあるけどよく知らん」とか
「レトロパイ？なら入れたことがある」くらいの人が、
「ディスプレイとキーボード無しでOSインストール」して「BOINCをインストールする」ところまでを
メモしたものです

[TOC]

## ・Wi-Fiアダプタの確認

Wi-Fiのアダプタがあることを確認

```
$ ls /sys/class/net
```

結果が「eth0  lo  wlan0」となる筈
この内の「wlan0」がWi-Fiアダプタ

## ・設定ファイルの編集

/etc/netplan/50-cloud-init.yamlを編集

```
$ sudo nano /etc/netplan/50-cloud-init.yaml
```

編集例

    network:
        ethernets:
            eth0:
                dhcp4: false
                optional: true
                addresses:
                    - 192.168.0.？/24
                gateway4: 192.168.0.？
                    nameservers:
                        addresses:
                            - 192.168.0.？
    
        wifis:
            wlan0:
                addresses:
                    - 192.168.0.？/24       <- Wi-Fiアダプタに割り当てるIPアドレス
                gateway4: 192.168.0.？
                nameservers:
                    addresses:
                        - 192.168.0.？
                dhcp4: false
                optional: true              <- ここまでは「eth0」セクションと同じ
                access-points:
                    singleboard:            <- アクセスポイント名
                        password: team5ch   <- アクセスポイントに対するパスワード
    
        version: 2 

2. 設定の反映

```
$ sudo netplan apply
$ sudo reboot
```

## ・IPアドレスの確認

再起動後、設定したIPアドレスに有効になっていることを確認する(例えばpingコマンド)

```
$ ping 192.168.0.？
```

「要求がタイムアウトしました。」「宛先ホストに到達できません。」とエラーが出なければ成功
設定したIPアドレスに対してssh接続して作業可能
