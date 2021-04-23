# ゼロくらいから始めるBOINC

このテキストは「ラズパイって聞いたことはあるけどよく知らん」とか
「レトロパイ？なら入れたことがある」くらいの人が、
「ディスプレイとキーボード無しでOSインストール」して「BOINCをインストールする」ところまでを
メモしたものです

[TOC]

## 1.ラズパイを買う

### ・本体

Amazonとか秋月とかRSコンポーネンツとかお好きなところでポチる  
電源とかまるっと一式入ってるやつがお手軽だと思う  
以降「ラズベリーパイ4B 8GB」を買う or 買ったものだとする
> 2021年4月中旬から4GBではタスクが振られない現象が起きているぽいので新しく買うなら16GBの方が無難

### ・ケース

4Bは発熱がでかいので小さいヒートシンクだけだと火傷しそうな勢いで熱くなる  
冷却ファン付きのケースだとか、ごついヒートシンクで全体を包むやつがお勧め

### ・microSD

16GB以上で高速なやつ。というか8GBとか最近見かけないし  
64GBとかでも値段は数百円しか変わらないので適当に容量でかいやつをポチる  
速度はA1/A2規格とかclass10とか。価格.comにも転送速度の情報載ってるから参考に
> Ubuntu 20.10でUSBストレージから起動できるようになったけどBOINCの起動でエラーが出て使えないので20.04を推奨 


### ・ケーブル類

電源供給用にUSB-Type Cコネクタの奴とLANケーブル  
今回は使わないけど、ディスプレイ繋いで作業する場合はmicro HDMIケーブルも  
HDMI→micro HDMI変換アダプタは電源ポートと干渉するのでお勧めしない
> うちではアーマーケースって奴使ってるけど、基盤全部を金属で囲うせいかWi-Fi接続がクッソ遅くなる  
> 冷却ファンは「Pi Fan」で検索して出てきたものをねじ止めして3V稼働。これで稼働中の温度は47度前後



## 2.OSをインストールする

### ・起動用microSDを作る

今回は64bit版のubuntu 19.10をインストール  
microSDカードにOSイメージを焼くのには「Raspberry Pi Imager」ってソフトを使用  
書き込み対象のmicroSDを指定して使いたいOS指定するだけ  
同じようなソフトに「noob」とか「balena Etcher」とか「DD for Windows」とかあるので好きなやつで

### ・ラズパイの起動

ラズパイにLANケーブル繋いでmicroSD挿して電源につなぐ  
最初はLANケーブル繋いでないとだめ  
電源スイッチ的なものは無いので、電源に繋ぐとそのまま起動する  
勝手にOSのインストールが始まるので10分くらい待機  
基盤の緑のLEDが点滅してるのが消えっぱなしになったらインストール完了

### ・IPアドレスの確認

OSのインストールが終わったら、現時点でのIPアドレス(ルーターから自動で割り振られたIPアドレス)を確認する  
今回は「NetEnum」ってソフトを使用  
「IPアドレス 一覧 確認」ってgoogleで検索すると他にも手段はあるので好きな方法で

### ・sshで接続

上で見つけたIPアドレスに対してssh接続する  
ユーザー名とパスワードは両方とも **ubuntu**  
初回ログイン時にパスワード変更を要求されるので適宜変更



## 2+.OSをUSBストレージにインストールする場合のメモ

microSDは寿命的にはショボイだとかUSBストレージの方が高速だとかいうのでやってみた  
ラズベリーパイ4BはUSBストレージから直接起動できないので少し搦め手を使う  
microSDからブートして、USBストレージを/パーティションとしてマウントする  

### ・起動用メディアを作る

前述のRaspberry Pi Imagerとかで同じOSを指定してmicroSDとUSBストレージの2つに焼く

### ・ラズパイの起動

microSDとUSBストレージの両方を繋いでラズパイを起動する  
起動(インストール)後、USBストレージが認識されているか確認

```
$ sudo fdisk -l
```

「/dev/mmcblk？？？」がmicroSD、「/dev/sd？？？」がUSBストレージ  
「sda」とか「sdb」とかは挿したUSBポートで変わる(以降sdaと仮定)  
「Disk identifier」の項目の値が同じのはず

### ・PARTUUIDを変更

fdiskからUSBストレージを操作する

```
$ sudo fdisk /dev/sda
```

以後、以下の通りキー押下

```
p -> x -> i -> (一意のPARTUUID) -> r -> w
```

```
--実画面--

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p
Disk /dev/sda: 59.6 GiB, 64016220160 bytes, 125031680 sectors
Disk model: Cruzer Fit
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x04c043ec

Device Boot Start End Sectors Size Id Type
/dev/sda1 8192 532479 524288 256M c W95 FAT32 (LBA)
/dev/sda2 532480 125031679 124499200 59.4G 83 Linux

Command (m for help): x

Expert command (m for help): i

Enter the new disk identifier: 97709164

Disk identifier changed from 0x04c043ec to 0x05d2ec6c.

Expert command (m for help): r

Command (m for help): w
The partition table has been altered.
Syncing disks.

----
```

変更後、PARTUUIDが変更されているか確認する

```
$ sudo blkid
```

「/dev/sda1」と「/dev/sda2」のPARTUUIDがそれぞれ「変更したPARTUUID-01」、「変更したPARTUUID-02」になっているはず

### ・/パーティーションの指定

「/boot/firmware/btcmd.txt」と「/boot/firmware/nobtcmd.txt」を編集

```
$ sudo nano /boot/firmware/btcmd.txt
$ sudo nano /boot/firmware/nobtcmd.txt
```
> Ubuntu20.04では「/boot/firmware/cmdline.txt」を編集する。「/boot/firmware/btcmd.txt」と「/boot/firmware/nobtcmd.txt」が無い

「root=云々」の部分を「root=PARTUUID=変更したPARTUUID-02」に書き換える

```
例) PARTUUIDが「abcd1234-02」の場合
root=PARTUUID=abcd1234-02
```

> USBストレージを挿すポートを完全に固定にする場合、PARTUUIDの変更をスキップして  
> 「root=云々」の部分を「root=/dev/sda2」に書き換えるだけでも大丈夫

再起動

```
$ sudo reboot
```

/パーティーションの確認

    $ findmnt -n -o SOURCE /
結果が「/dev/sda2」であれば成功



## 3.環境整備とかアプリ入れたりとか

### ・パッケージの更新

```
$ sudo apt-get update
$ sudo apt-get upgrade -y
```

### ・IPアドレスの固定

1. 「/etc/netplan/99-cloud-init.yaml」というファイルを新規作成
> 「/etc/netplan/50-cloud-init.yaml」を編集するのは悪手なので新たに定義ファイルを作るのが正しいらしい
```
$ sudo nano /etc/netplan/99-cloud-init.yaml
```

編集例

    network:
        ethernets:
            eth0:
                dhcp4: false            <- DHCPクライアント機能の無効化
                optional: true
                addresses:
                    - 192.168.0.？/24    <- ラズパイのIPアドレス
                gateway4: 192.168.0.？    <- ゲートウェイのIPアドレス
                nameservers:
                    addresses:
                        - 192.168.0.？    <- DNSサーバのIPアドレス
        version: 2 
2. 設定の反映。コマンド実行後は設定したIPアドレスになるので、改めてそのアドレスにssh接続する

```
$ sudo netplan apply
```

### ・swap領域の確保

1. 実メモリの2倍の値を設定するのが一般的らしいので16GBを指定

```
$ sudo fallocate -l 16G /swapfile
$ sudo chmod 600 /swapfile
$ sudo mkswap /swapfile
```

2. /etc/fstabを編集

```
$ sudo nano /etc/fstab
```

 どこでも良いので以下の1行を追加

```
/swapfile none swap sw 0 0
```

### ・SSH接続時の日本語環境構築

1. 日本語関連パッケージのインストール

```
$ sudo apt-get install -y language-pack-ja-base
$ sudo apt-get install -y language-pack-ja
```

2. 文字セットを日本語に変更

```
$ sudo localectl set-locale LANG=ja_JP.UTF-8 LANGUAGE="ja_JP:ja"
```

3. タイムゾーンを日本にする

```
$ sudo timedatectl set-timezone Asia/Tokyo
```

4. 再起動

```
$ sudo reboot
```

### ・BONICのインストール

1. BOINCのクライアントとマネージャーをインストール

```
$ sudo apt-get install -y boinc-client
$ sudo apt-get install -y boinctui
```
> 「boinc-manager」というGUIのI/Fも同じ手順でインストールできるけど使わないのでスキップ

2. スレのテンプレを参考にプロジェクトへ参加。お疲れさまでした



## 4.コマンド色々

| 実行内容                                  | コマンド                                  |
| ----------------------------------------- | ----------------------------------------- |
| BOINCを操作したい・ステータスを確認したい | boinctui                                  |
| CPUとメモリの使用率を見たい               | top , htop                                |
| メモリの使用状況を見たい                  | free -m                                   |
| 温度を見たい                              | cat /sys/class/thermal/thermal_zone0/temp |

