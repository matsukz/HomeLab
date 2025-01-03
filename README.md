# 楽しい自宅サーバー
自宅サーバーについての解説とメモです。
## 最新図！
https://github.com/matsukz/HomeLab-Renew

ツッコミどころ満載ですがご容赦ください。

## 目次
- [楽しい自宅サーバー](#楽しい自宅サーバー)
  - [目次](#目次)
  - [なぜこんなことを始めたのか](#なぜこんなことを始めたのか)
  - [全体図](#全体図)
  - [物理機器](#物理機器)
    - [TP-LINK ArcherAX73](#tp-link-archerax73)
    - [BUFFALOのやつ](#buffaloのやつ)
    - [Let'snote](#letsnote)
    - [新デスクトップ](#新デスクトップ)
  - [仮想マシン・サービス](#仮想マシンサービス)
    - [検証用Ubuntu](#検証用ubuntu)
    - [VPN DNS](#vpn-dns)
    - [WARP接続用サーバー](#warp接続用サーバー)
    - [NextCloud](#nextcloud)
    - [Pythonスクリプト](#pythonスクリプト)
    - [Webサーバー](#webサーバー)
  - [今後の展望](#今後の展望)
  - [まとめ](#まとめ)
  - [クレジット](#クレジット)

## なぜこんなことを始めたのか
初心忘るべからず

### 2019年
高校１年生のとき、友人からのプレゼントで128GBのSSDを貰いました。

そのSSDにPCやスマホから接続しようと、中古のラズパイ3でNASを構築したのがエンジニアごっこの始まりです。

NASを運用するにつれ、次のようなことを考え始めます

* ラズパイ3の性能ではネットワークや電源周りが不安定
* 外出先からでも接続できるようにしよう
* Pi-HoleというDNSサーバー(後述)があるらしい
* 自作のPythonスクリプトを動かしたい
* Linuxサーバーで遊びたいが、壊してもいいマシンが無い

### 2021年
メルカリで中古のデスクトップPCを見つけて購入しました。

元々はWindows7を搭載していたので、スペックは中の下ぐらいです。

それにUbuntu Desktopをインストールし自宅サーバーとして運用していました。

OpenVPN、DNS、Samba、Pythonなどのサービスを運用するサーバーとして利用していたのですが、`ひとつトラブルが発生すると他のサービスにまで影響を及ぼす`という問題を抱えていました。

あとRAMの認識が甘く、4GBしか認識できなくなった。

RDPもうまくいかないし保守性が終わってました。

### 2023年
ついに革命が起きます。

それは`Proxmox VE`という仮想化基盤を導入したことです。

* ブラウザで管理できる
* VMとLXCによりサービスを分離することができる
* VMやLXC丸々のバックアップ/リストアが可能
* サービスを停止させることなくホコリ掃除ができる(マイグレート)

これが無料ってマ？

VMが利用できるようになり、リスクを伴う検証が気軽にできるようになりました。

あれこれ考えるの本当に楽しいです。

## 全体図
![Map](img/ServerMap2.png)

Proxmoxをインストールしたマシンを２台用意しクラスタを形成しています。

ただしLet'snoteは入院中なため、デスクトップPC１台で運用しています。

外部に公開するサーバーはすべてCloudflareを経由しています
![zero](img/cloudflarezero.png)
## 物理機器
ここからは自分用のメモを含んでいます。

### TP-LINK ArcherAX73
#### 分類
* ネットワーク機器
#### 役割
* 家族全員で利用している無線LANルーター
* ISPに繋がっている
### BUFFALOのやつ
#### 分類
* ネットワーク機器
#### 役割
* Archerへの負荷を軽減するため、検証系や速度を要求しないサーバーまとめる
  
### Let'snote
#### 分類
* 物理ノード
#### 役割
* Proxmoxを実行する
* メインのデスクトップPCをメンテナンスするときにVMを移動させサービスを継続する
#### メモ
* 故障したためクラスタから切断したが、近日復帰予定
#### リンク集
* ノード切断で困っていたときにした質問
* [Unable to remove a node from the cluster. - reddit](https://www.reddit.com/r/Proxmox/comments/196enuk/unable_to_remove_a_node_from_the_cluster/)

### 新デスクトップ
#### 分類
* 物理ノード
#### 役割
* Proxmoxの実行
* 肝　心臓　命　死守
## 仮想マシン・サービス
Proxmoxで実行している仮想マシン等です

### 検証用Ubuntu
#### 分類
* VM
#### システム
* Ubuntu Server x64
* docker-compose
#### 役割
* 様々な検証をするやつ
* 壊しすぎて何度リストアしたか

### VPN DNS
#### 分類
* VM
#### システム
* Ubuntu Server x64
#### 役割
* 自宅サーバーへVPN接続するOpenVPNサーバー
* 広告ブロッカー[Pi-hole®](https://pi-hole.net/)を実行するDNSサーバー
#### OpenVPN
* [PiVPN](https://www.pivpn.io/)というソフトウェアを使用しています
* ただのVPNサーバーですが、事情があり変な形になっています
    * [詳細 - VPNServer.md](Supplementary/VPNServer.md)
    * 最も労力をかけたサーバーなのかもしれません
#### DNSサーバー
* 広告ブロッカーとして動作するローカルDNSサーバーです
* 円安時でもドネートしてしまうぐらい最高です

#### 不正出席！？
UNIPAで出席登録を行うとき、VPNを有効にしていると教室に居ない判定されます。

UNIPAへの接続元IPアドレスで教室に居るのか居ないのかを判別していると思います。

~~迷惑~~ 不正出席を防ぐ素晴らしいシステムです。

#### 対策
VPN接続時はUNIPAのドメインを解決しないようにする
  * Pi-holeのグループ機能を利用

### WARP接続用サーバー
#### 分類
* VM
#### システム
* Ubuntu Server x64
#### 役割
* Cloudflare社が提供しているサービス`WARP-to-WARP`で構築したネットワークに、ローカルから接続するためのNAT要員
  * [Weave your own global, private, virtual Zero Trust network on Cloudflare with WARP-to-WARP - The Cloudflare Blog](https://blog.cloudflare.com/warp-to-warp)
  * 
* 遠隔地に存在する機器にローカルに存在するかのようなノリでアクセスできる
![warp](img/warp-to-warp2.png)

### NextCloud
#### 分類
* VM
#### システム
* Ubuntu Desktop x64
  * 日本語ディレクトリの文字化け防止
* Snap版NextCloud
#### 役割
* NextCloudでファイルサーバーを運用する
* 外部からの接続にはCloudflare Tunnelを利用する
![Nextcloud](img/NextCloud.png)
* 設定ファイル
  * [Nginx Configuration - nextcloud.conf](Supplementary/nextcloudc.conf)
  * [NextCloud Configuration - config.php](Supplementary/config.php)

#### トラブル
##### CloudflaredとNginxを挟むとドメインが信頼できなくなる
* 原因
  * 接続元IPアドレスに関するHTTPヘッダが`Cloudflared`もしくは`Nginx`のIPアドレス`127.0.0.1`になっている
* 解決
  * Cloudflare CDNを経由したときに付与されるヘッダ`CF-Connecting-IP`に関する設定を行う
    ```config
    set_real_ip_from 127.0.0.1;
    real_ip_header CF-Connecting-IP;
    ```
  * [Restoring original visitor IPs - Cloudflare docs](https://developers.cloudflare.com/support/troubleshooting/restoring-visitor-ips/restoring-original-visitor-ips/)
  * スキーマに関する設定を追加する
    ```config
    proxy_set_header X-Forwarded-Proto $scheme
    ```
##### SSDへの書き込みができない
* 原因
  * 権限？
* 解決
  * SFTP経由で読み書きする

### Pythonスクリプト
#### 分類
* LXC
#### システム
* ubuntu-22.04-standard_22.04-1_amd64
#### 役割
* UNIPAで出席登録が開始されたことを通知するスクリプトを動かす
  * [ClassNotice - GitHub](https://github.com/matsukz/ClassNotice)
### Webサーバー
#### 分類
* LXC
#### システム
* ubuntu-22.04-standard_22.04-1_amd64
* Docker Engine
* Docker Compose
#### 役割
* サーバーの死活確認を行うWebサーバーを実行する
  * [Check_Server_Health - Github](https://github.com/matsukz/Check_Server_Health)
  * 意図しないアクセスを防ぐため、Cloudflare Tunnelの認証機能を利用
    * 条件はヒ・ミ・ツ

## 今後の展望
### サーバー
* Proxmoxノードを増やしてクラスタを大きくしたい
  * 冗長化のためとかロマンとかロマンとか
* ストレージ障害へ対策をしたい
  * HAとかCephっていう分散ストレージがあるらしい
  * NASを導入してライブマイグレーションができるようにしたい

### ネットワーク
現在は家族共有の無線LANルーターを利用していますが、今後はVLANの設定ができるネットワーク機器を導入したいと考えています。
* 例：YAMAHA RTX
* 例：NEC UNIVERGE IXシリーズ

## まとめ
自宅サーバーは楽しいです。無限の可能性があります。

特にProxmoxが使えるようになったのは革命的です。壊してもいい環境があると気軽に勉強ができます。

趣味とお勉強が同時にできて便利です。

また自宅サーバーを拡張するためだと考えるとアルバイトをするモチベーションも保てます。

一石二五六鳥ですね。

<br>

ありがとうございました。

2024-01-21

## クレジット
* 図作成
  * draw.io
  * Microsoft PowerPoint
* 画像利用
  * さくらのアイコンセット
    * https://knowledge.sakura.ad.jp/4724/
  * Cloudflare プレスキット
    * https://www.cloudflare.com/ja-jp/press-kit/
  * NGINXのロゴ
    * https://en.wikipedia.org/wiki/File:Nginx_logo.svg
