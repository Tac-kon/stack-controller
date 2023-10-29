# Kubernetes クラスタへの接続

以降の手順では、Kubernetes クラスタの状態を確認したり、クラスタを操作する際に`kubectl`コマンドを実行することになります。

[クラスタ作成の手順](./create-cluster.md)では、コントローラ 1 にログインして`kubectl get nodes`を実行してクラスタ内のノードを確認する、という手順があります。

これについては作業用 PC から`kubectl`コマンドを実行できた方が何かと便利なので、今回はそのための設定を行いたいと思います。

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

作業用 PC からリモートの Kubernetes クラスタにアクセスするには Kubernetes クラスタの認証情報を作業用 PC に登録する必要があります。

下記のコマンドを実行して、作業用 PC の`~/.kube/config`にコントローラ 1 から読みだしたクラスタの認証情報を書き込みます。

```
CONFIG=${HOME}/.kube/config
mkdir ${HOME}/.kube
touch ${CONFIG}

CP1_ADDR=10.0.1.2
echo "$(ssh ubuntu@${CP1_ADDR} kubectl config view --raw) | tee -a ${CONFIG}
sed -e 's/127.0.0.1:6443/${CP1_ADDR}:6443/g' ${CONFIG}
sed -e 's/cluster.local/cluster.dc/g' ${CONFIG}
```

### 動作確認

作業用 PC 上で`kubectl get nodes`を実行します。下記のようにクラスタ内のサーバー一覧が表示されれば設定は完了です。

```
 kubectl get nodes
NAME   STATUS   ROLES           AGE     VERSION
cp1    Ready    <none>          5d10h   v1.26.5
cp2    Ready    <none>          5d10h   v1.26.5
cp3    Ready    <none>          5d10h   v1.26.5
cr1    Ready    control-plane   5d10h   v1.26.5
cr2    Ready    control-plane   5d10h   v1.26.5
cr3    Ready    control-plane   5d10h   v1.26.5
```

### 参考

- https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/
- https://qiita.com/ytyng/items/0d9cbfda8ecb985d5858
