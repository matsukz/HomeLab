# VPNサーバーについて

至って普通のOpenVPNサーバーですが、ある事情から変な構成になっています。

## 「ある事情」
ずばりファイアウォールです。

大学のネットワーク内から自宅にVPN接続したくなることがあります。

* ProxmoxのコントロールパネルやSSHに接続するとき
* Pi-Holeの機能を利用したいとき
* 自宅のグローバルIPアドレスが必要なとき
* 検閲対策

しかし組織のFWは厳密に設定されていて、特定サービス以外はインターネットに出ることができません。

![vpn0](../img/vpn0.png)

* 例：リッスンポート変更したSSHサーバに接続できない
* 例：プロジェクトセカイのマルチプレイに接続できない

当然ながらOpenVPNのデフォルトポートである`1194/UDP`も規制されているようで、VPNサーバーに到達できません。

そこで、HTTPSを利用してVPN接続を行うことにしました。

## 変更１ HTTPSを利用する
組織のネットワークといえど普通にネットサーフィンが可能です。

つまりHTTPS(443/TCP)に向けた通信は可能ではないかと考えました。

検閲が厳しい国やVPNが禁止されている国でよく行われる手法です。

![vpn1](../img/vpn1.png)

しかしこの方法だと次のような問題が発生します
* HTTPSに対応したWebページの公開ができない
  * OpenVPNが443/TCPを利用するため
* そもそもHTTPSを開放するのは危険
  * GCPで公開していたWebサーバーは多数のインジェクションを受け閉鎖

## 変更２ VPSを経由する
NGINXによるTCPプロキシサーバーを建てました。

自宅の443/TCPを開放するのではなく、別のサーバーで443/TCPをリッスンし、ポートを書きかけて自宅に接続するようにしました。

![vpn2](../img/vpn2.png)

VPNサーバーのファイアウォールで接続元IPを絞ることでマシになります。

VPSのグローバルIPアドレスは静的です。
```bash
$ sudo ufw allow from VPSのIP to any port 443
```

自宅の443/TCPは閉鎖できましたが、すべてのポートを閉鎖したいと考えていました。

意図しない通信を自宅サーバーに入れたくないです。

その悩みを解決してくれたのが`Cloudflare ZeroTrust`です。

## 現状 Cloudflareを挟む
Cloudflare ZeroTrustの中で`WARP`というサービスを発見しました。

WARPとはCloudflare社の`1.1.1.1`へ接続することができるいい感じなVPNツールですが、チームを作成し独自のネットワークを作成することが可能です。

* [Weave your own global, private, virtual Zero Trust network on Cloudflare with WARP-to-WARP - The Cloudflare Blog](https://blog.cloudflare.com/warp-to-warp)

* [Cloudflare Zero Trust 経由でAWS上のEC2（グローバルIPアドレス無し）にSSHでログインする - Zenn](https://zenn.dev/hiroe_orz17/articles/b028fdb5444ee0)

WARPでチームを作成し、VPSと自宅を接続すれば安全に遠隔地と通信することができます。

またNGINX上でフィルタを設定すると、意図しない接続は自宅サーバーへ届きません。

![vpn3](../img/vpn3.png)

Cloudflareのサーバーとコネクションを張るため、自宅ポートの開放が不要になりました。

## まとめ
Cloudflareは神

以上です。