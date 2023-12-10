# Kubernetes クラスタへの接続
以降の手順ではKubernetesクラスタの状態を確認したり、クラスタを操作する際に`kubectl`コマンドを実行します。
例えば[クラスタ作成の手順](./create-cluster.md)では、コントローラ1で`kubectl get nodes`を実行してクラスタ内のノードを確認しています。

しかし、都度作業用PCとコントローラーノードのターミナルを切り替えるのは手間がかかります。
そこで作業用PCから`kubectl`コマンドを実行できるように設定をします。

## kubectl のインストール
まずは作業用 PC に`kubectl`をインストールします。

```
KUBECTL_VERSION=1.28.0
curl -LO https://dl.k8s.io/release/v${KUBECTL_VERSION}/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --client
```

正しいバージョンが出力されていれば、正常にインストールされています。

## Kubernetes クラスタへの接続設定
### 認証情報を作業用 PC に登録
作業用PCからリモートのKubernetesクラスタにアクセスするには、クラスタの認証情報を作業用PCに登録する必要があります。

下記のコマンドを実行して、作業用 PC の`~/.kube/config`にコントローラ1から読みだしたクラスタの認証情報を書き込みます。
既に`~/.kube/config`が存在する場合は、別名にするなどして既存のファイルが上書きされないようにしてください。

```
CONFIG=${HOME}/.kube/config
mkdir ${HOME}/.kube
touch ${CONFIG}

CP1_ADDR=10.0.1.2
echo "$(ssh ubuntu@${CP1_ADDR} kubectl config view --raw) | tee ${CONFIG}
sed -ie 's/cluster\.local$/cluster\.dc/g' ${CONFIG}
```

### HAProxyのインストール
下記のように、この時点では作業用PCにて`kubectl`コマンドを実行するとローカルホスト(= 127.0.0.1:6443)に接続する設定になっています。
```
kubectl config view -o json | jq -r '.clusters[0].cluster.server'
https://127.0.0.1:6443
```

上記の接続を各コントローラノードにプロキシさせるために**HAProxy**を使用します。
下記のコマンドで作業用PCにHAProxyをインストールします。
```
sudo apt update
sudo apt -y install haproxy
```

インストールが完了したら、下記のようにしてHAProxyの設定を行います。
下記ではローカルホストの6443ポートへの通信が、いずれかのコントローラノードの6443ポートにプロキシされるように設定しています。
```
sudo tee -a /etc/haproxy/haproxy.cfg <<EOF
frontend kube_api
    bind 127.0.0.1:6443
    mode tcp
    default_backend kube_cr

backend kube_cr
    mode tcp
    balance    roundrobin
    server cr1 10.0.1.2:6443 check port 6443
    server cr2 10.0.1.3:6443 check port 6443
    server cr3 10.0.1.4:6443 check port 6443

sudo systemctl restart haproxy
```

エラーなどが出なければ、これにてHAProxyの設定は完了です。

### 動作確認
作業用PC上で`kubectl get nodes`を実行します。下記のようにクラスタ内のサーバー一覧が表示されれば設定は完了です。
```
kubectl get nodes
NAME   STATUS   ROLES           AGE    VERSION
cp1    Ready    <none>          112m   v1.26.5
cp2    Ready    <none>          112m   v1.26.5
cp3    Ready    <none>          112m   v1.26.5
cr1    Ready    control-plane   169m   v1.26.5
cr2    Ready    control-plane   169m   v1.26.5
cr3    Ready    control-plane   168m   v1.26.5
st1    Ready    <none>          112m   v1.26.5
st2    Ready    <none>          112m   v1.26.5
st3    Ready    <none>          112m   v1.26.5
```

### 参考

- https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/
- https://qiita.com/ytyng/items/0d9cbfda8ecb985d5858
- https://qiita.com/yoshi_1010/items/6563fe70897a552e1d84
