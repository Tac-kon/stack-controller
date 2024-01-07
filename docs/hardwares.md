# 物理機材
Kubernetesクラスタのノード用機材やネットワーク機器など、システムを構築するために使用した物理機材について紹介します。

## ノード機材
下記3種類のノード用の機材を使用しました。

### 1. Controller node
OpenStackの中核となる各種サービスをこのノード上で動かします。具体的には下記のようなサービスがあります。
 - keystone：ユーザー認証を行う。
 - glance：仮想イメージを管理する。
 - nova：インスタンスの作成等を担う。
 - neutron：仮想ネットワークを管理する。

また、KubernetesのControl planeとしても使用します。

[Raspberry Pi4 ModelB 4GB](https://www.amazon.co.jp/dp/B081YD3VL5)を使用しました。
また、電源供給は[PoE HAT](https://www.amazon.co.jp/gp/product/B082LR3DDN)経由で行うようにしています。

### 2. Storage node
OpenStackのインスタンスに提供するストレージシステムをこのノード上で動作させます。

具体的にはRook-Cephという分散ストレージシステムをこのノード上で動かし(詳細は後述)、各インスタンスのデータがこのストレージシステムに保管されるようにします。

[GMKtec ミニPC](https://www.amazon.co.jp/gp/product/B0C22Z6P8R)を使用しました。

### 3. Compute node
OpenStackのインスタンスを動かすノードです。
[SkyBarium ミニPC](https://www.amazon.co.jp/gp/product/B0B7DLK42Q)を使用しました。

## ネットワーク機器
ネットワーク機器には下記を使用しました。
 - ルータ
    - [YAMAHA RTX 810](https://www.amazon.co.jp/gp/product/B005TC9B7M)：外部アクセスやセグメント分離を再現でき、かつ比較的安価なものを選定。
 - L2スイッチ
    - [NETGEAR スイッチングハブ 8ポート](https://www.amazon.co.jp/gp/product/B07S9X5NW9)：ルータ <-> 各ノード機材間をつなぐ用途で使用。
    - [NETGEAR スイッチングハブ 5ポート 1G PoE+](https://www.amazon.co.jp/gp/product/B08SQ3QZBH)：Raspberry Pi用。PoEの電源供給が可能。

## 物品一覧
上記機材・機器も含め、筆者が今回用意した物品の一覧を紹介します。

| TH 左寄せ | TH 中央寄せ | TH 右寄せ |
| :--- | :---: | ---: |
| TD | TD | TD |
| TD | TD | TD |


| 品目 | 用途 | 個数 | 備考 |
| --- | --- | --- | --- |
| [Raspberry Pi4 ModelB 4GB](https://www.amazon.co.jp/dp/B081YD3VL5) | Kubernetesノード (Control node) | 3 |
| [GeeekPi Raspberry Pi 4用アクリルケース付きPoE HAT](https://www.amazon.co.jp/gp/product/B097NCD6FW) | Raspberry Piの給電 | 3 |
| [SanDisk microSD 32GB](https://www.amazon.co.jp/gp/product/B08GY9NYRM) | Raspberry PiのOSインストール用 | 3 |
| [GMKtec ミニPC](https://www.amazon.co.jp/gp/product/B0C22Z6P8R) | Kubernetesノード (Storage node) | 3 |
| [SkyBarium ミニPC](https://www.amazon.co.jp/gp/product/B0B7DLK42Q) | Kubernetesノード (Compute node) | 3 |
| [YAMAHA RTX 810](https://www.amazon.co.jp/gp/product/B005TC9B7M) | ルータ | 1 |
| [NETGEAR スイッチングハブ 8ポート](https://www.amazon.co.jp/gp/product/B07S9X5NW9) | L2スイッチ | 1 |
| [NETGEAR スイッチングハブ 5ポート 1G PoE+](https://www.amazon.co.jp/gp/product/B08SQ3QZBH) | L2スイッチ | 1 | Raspberry Piの給電も兼務
| [10Gtek LANケーブル 0.5m 10本入り](https://www.amazon.co.jp/gp/product/B09JGCYDBX) | LANケーブル | 1 |
| [Lasocy ケーブルクリップ 20本入り](https://www.amazon.co.jp/gp/product/B0BWYCDLHV) | ケーブルの収納 | 1 |
| [エレコム 電源タップ 10個口](https://www.amazon.co.jp/gp/product/B08776L9W6) | 各機材の給電 | 1 | ミニPCのバッテリーのサイズによっては差込口の感覚が狭い可能性があるので注意
| [HUANUO 机上ラック](https://www.amazon.co.jp/gp/product/B07PV8RCG8) | 各機材の収納 | 1 | 機材のバッテリーのスペース等を考慮すると、幅が1.5倍程度あるものを選んでも良かった
