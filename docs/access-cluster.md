# Kubernetesクラスタへの接続
以降の手順では、Kubernetesクラスタの状態を確認したり、クラスタを操作する際に`kubectl`コマンドを実行することになります。

[クラスタ作成の手順](./create-cluster.md)では、コントローラ1にログインして`kubectl get nodes`を実行してクラスタ内のノードを確認する、という手順があります。

これについては作業用PCから`kubectl`コマンドを実行できた方が何かと便利なので、今回はそのための設定を行いたいと思います。

## kubectlのインストール
まずは作業用PCに`kubectl`をインストールします。
```
user@local-pc:~$ KUBECTL_VERSION=1.28.0
user@local-pc:~$ curl -LO https://dl.k8s.io/release/v${KUBECTL_VERSION}/bin/linux/amd64/kubectl
user@local-pc:~$ chmod +x ./kubectl
user@local-pc:~$ sudo mv ./kubectl /usr/local/bin/kubectl
user@local-pc:~$ kubectl version --client
```
正しいバージョンが出力されていれば、正常にインストールされています。

## Kubernetesクラスタへの接続設定

### 認証情報を作業用PCに登録
作業用PCからリモートのKubernetesクラスタにアクセスするにはKubernetesクラスタの認証情報を作業用PCに登録する必要があります。

下記のコマンドを実行して、作業用PCの`~/.kube/config`にコントローラ1から読みだしたクラスタの認証情報を書き込みます。
```
user@local-pc:~$ CONFIG=${HOME}/.kube/config
user@local-pc:~$ mkdir ${HOME}/.kube
user@local-pc:~$ touch ${CONFIG}

user@local-pc:~$ CP1_ADDR=10.0.1.2
user@local-pc:~$ echo "$(ssh ubuntu@${CP1_ADDR} kubectl config view --raw) | tee -a ${CONFIG}
user@local-pc:~$ sed -e 's/127.0.0.1:6443/${CP1_ADDR}:6443/g' ${CONFIG}
user@local-pc:~$ sed -e 's/cluster.local/cluster.dc/g' ${CONFIG}
```

### 動作確認
作業用PC上で`kubectl get nodes`を実行します。下記のようにクラスタ内のサーバー一覧が表示されれば設定は完了です。
```
user@local-pc:~$  kubectl get nodes
NAME   STATUS   ROLES           AGE     VERSION
cp1    Ready    <none>          5d10h   v1.26.5
cp2    Ready    <none>          5d10h   v1.26.5
cp3    Ready    <none>          5d10h   v1.26.5
cr1    Ready    control-plane   5d10h   v1.26.5
cr2    Ready    control-plane   5d10h   v1.26.5
st1    Ready    control-plane   5d10h   v1.26.5
```

### 参考
 - https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/
 - https://qiita.com/ytyng/items/0d9cbfda8ecb985d5858