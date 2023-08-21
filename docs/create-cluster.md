# Kuberntesクラスタ作成
Kubernetesクラスタを作成するのに、今回は[kubespray](https://github.com/kubernetes-sigs/kubespray)を使用します。
kubesprayを使用することで複数台のサーバーをまとめてクラスタ化することができます。

## kubespray実行
### 必要なパッケージのインストール
まずは作業用PCにてgithubからkubesprayのリポジトリをダウンロードして展開します。

```
user@local-pc:~/stack-controller$ cd
user@local-pc:~$ KUBESPRAY_VERSION=2.22.1
user@local-pc:~$ wget https://github.com/kubernetes-sigs/kubespray/archive/refs/tags/v${KUBESPRAY_VERSION}.tar.gz

user@local-pc:~$ mkdir ~/kubespray
user@local-pc:~$ tar -xvf v${KUBESPRAY_VERSION}.tar.gz -C kubespray --strip-components 1
user@local-pc:~$ rm v${KUBESPRAY_VERSION}.tar.gz
```

kubesprayの実行にはAnsibleを使用するため、必要なパッケージをインストールします。
```
user@local-pc:~$ cd ~/kubespray
user@local-pc:~/kubespray$ sudo apt -y install python3-pip
user@local-pc:~/kubespray$ sudo pip install -r requirements.txt
```

### インベントリファイルのコピー
続いて、このリポジトリ内のinventoryファイルをコピーします。
```
user@local-pc:~/kubespray$ cp -r inventory/sample inventory/cluster
user@local-pc:~/kubespray$ cp -r ~/stack-controller/kubespray ~/
```

ここでinventoryディレクトリ以下の、今回変更を加えた3つのファイルについて簡単に説明します。
 - group_vars/k8s_cluster/addons.yml: `MetalLB`を有効化するための設定を入れています。MetalLBとは、オンプレミスのKubernetes環境で`LoadBalancer`型のServiceリソースを有効化するソフトウェアです。今回は各種コントローラのサービスにLoadBalancerを使用するためにMetalLBの設定を入れます。
 - group_vars/k8s_cluster/k8s-cluster.yml: 上記の`MetalLB`を有効化するために、主にネットワークドライバに関する設定を入れています。
 - inventory.ini: インベントリファイルと呼ばれるファイルで、Kubernetesクラスタを構築する各サーバーの情報(IPアドレス、ホスト名など)が記載されたファイルです。今回使用するサーバーの情報に合わせて編集してあります。

### クラスタ作成
実際にクラスタを作成するために、下記のコマンドを実行します。
実行後、完了するまでに数10分~1時間程度かかるので、しばらく放置します。
```
user@local-pc:~/stack-controller/kubespray$ ansible-playbook -i inventory/cluster/inventory.ini cluster.yml -u ubuntu --become --private-key="~/.ssh/id_rsa"
```

コマンド内の個々のオプションについて簡単に説明します。
 - -i: インベントリファイルを指定します。
 - -u: 各サーバーにログインするためのユーザー名を指定します。
 - --become: root権限を付与します。一部のタスクを実行するために必要です。
 - --private-key: 各サーバーにSSH接続するための秘密鍵を指定します。

## 動作確認
kubesprayのAnsible完了後、クラスタが正常に構築されているか確認します。

今回はコントローラー1にログインして、クラスタの動作確認を行います。
まずは以下のコマンドを実行して、`kubectl`をユーザー権限で実行できるようにします。
```
ubuntu@cr1:~$ sudo cp -r /root/.kube ~/
ubuntu@cr1:~$ sudo chown -R ${USER}:${USER} ~/.kube
```

`kubectl get nodes`を実行します。正常にクラスタが構築されている場合、下記のようにサーバー一覧が表示されます。
```
ubuntu@cr1:~$ kubectl get nodes
NAME   STATUS   ROLES           AGE    VERSION
cp1    Ready    <none>          100m   v1.26.5
cp2    Ready    <none>          100m   v1.26.5
cp3    Ready    <none>          100m   v1.26.5
cr1    Ready    control-plane   102m   v1.26.5
cr2    Ready    control-plane   101m   v1.26.5
st1    Ready    control-plane   101m   v1.26.5
ubuntu@cr1:~$ 
```

### 参考
 - https://github.com/kubernetes-sigs/kubespray
 - https://qiita.com/suzuyui/items/e7531fe5e1e84c061b23