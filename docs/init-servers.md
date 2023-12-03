# サーバーの初期セットアップ
各サーバーにてOSインストール後に以下のセットアップ作業を行います。


## 公開鍵設定
まずは作業用PCにて下記のコマンドを実行して、SSH接続用の秘密鍵/公開鍵のキーペアを作成します。
```
ssh-keygen -t rsa
```

いくつかの項目にてプロンプトが表示されますが、すべて「Enter」を入力します。
キーペア作成後、作業用PCにて各サーバーに設定する公開鍵の中身を確認しておきます。
```
cat ~/.ssh/id_rsa.pub
ssh-rsa ~~~ user@local-pc  ## この出力結果を各サーバーに設定する。
```

続いて、セットアップ対象のサーバーにログインして、先ほど作成したSSH公開鍵を設定します。
```
ssh ubuntu@(サーバーのIPアドレス)
PUBLICK_KEY="( cat ~/.ssh/id_rsa.pub の出力結果 )"
echo "${PUBLICK_KEY}" | tee -a ${HOME}/.ssh/authorized_keys
```

公開鍵設定後、作業用PCからパスワード入力なしでSSH接続できるか確認します。
```
ssh ubuntu@(サーバーのIPアドレス)
```
パスワードを聞かれずにSSH接続できたら設定完了です。

## sudoのパスワード無効化
サーバー内で下記コマンドを実行して、sudo権限を付与したコマンドをパスワード入力なしで実行できるようにします。
```
echo "ubuntu ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/99-config
```

## netplan編集
サーバー内で下記コマンドをしてネットワークの設定を行います。

もしもセットアップに失敗してSSH接続できなくなった場合は直接ログインして`/etc/netplan/99-config.yaml`の中身を確認して設定しなおしてください。

```
sudo rm /etc/netplan/*.yaml

NIC=$(for DEV in `find /sys/devices -name net | grep -v virtual`; do ls $DEV/ | grep -e "enp" -e "eth"; done)
IP_ADDRESS=$(ip addr show | grep 10.0.1. | awk {'print $2'})

sudo tee /etc/netplan/99-config.yaml <<EOF
network:
  ethernets:
    ${NIC}:
      dhcp4: false

  vlans:
    vlan.100:
      id: 100
      link: ${NIC}
      addresses:
      - ${IP_ADDRESS}
      optional: false
      routes:
      - to: default
        via: 10.0.1.1
      nameservers:
        addresses:
        - 10.0.1.1
        search: []

    vlan.200:
      id: 200
      link: ${NIC}
      dhcp4: false
      optional: false
  version: 2
EOF

sudo netplan apply
```
