# Kuberntes クラスタ作成

Kubernetes クラスタを作成するのに、今回は[kubespray](https://github.com/kubernetes-sigs/kubespray)を使用します。
kubespray を使用することで複数台のサーバーをまとめてクラスタ化することができます。

## kubespray 実行

### 必要なパッケージのインストール
まずは作業用PCにてgithubからkubesprayのリポジトリをcloneします。

```
KUBESPRAY_VERSION=2.22.1
git clone --depth 1 -b v${KUBESPRAY_VERSION} https://github.com/kubernetes-sigs/kubespray.git
```

kubespray の実行には Ansible を使用するため、必要なパッケージをインストールします。

```
cd ~/kubespray
sudo apt -y install python3-pip
sudo pip install -r requirements.txt
```

### インベントリファイルのコピー

続いて、このリポジトリ内の inventory ファイルをコピーします。

```
cd ~/kubespray
cp -r inventory/sample inventory/cluster
cp -r ~/stack-controller/kubespray ~/
```

ここで inventory ディレクトリ以下の、今回変更を加えた 2 つのファイルについて簡単に説明します。

- group_vars/k8s_cluster/k8s-cluster.yml: 今回は[MetalLB](https://metallb.universe.tf/)を有効化するために、ネットワークドライバを[Cilium](https://cilium.io/)にする設定を入れています。
- inventory.ini: インベントリファイルと呼ばれるファイルで、Kubernetes クラスタを構築する各サーバーの情報(IP アドレス、ホスト名など)が記載されたファイルです。今回使用するサーバーの情報に合わせて編集してあります。

### クラスタ作成

実際にクラスタを作成するために、下記のコマンドを実行します。
実行後、完了するまでに数 10 分~1 時間程度かかるので、しばらく放置します。

```
cd ~/kubespray
ansible-playbook -i inventory/cluster/inventory.ini cluster.yml -u ubuntu --become --private-key="~/.ssh/id_rsa"
```

コマンド内の個々のオプションについて簡単に説明します。

- -i: インベントリファイルを指定します。
- -u: 各サーバーにログインするためのユーザー名を指定します。
- --become: root 権限を付与します。一部のタスクを実行するために必要です。
- --private-key: 各サーバーに SSH 接続するための秘密鍵を指定します。

## 動作確認

kubespray の Ansible 完了後、クラスタが正常に構築されているか確認します。

今回はコントローラ 1 にログインして、クラスタの動作確認を行います。
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
cr3    Ready    control-plane   101m   v1.26.5
ubuntu@cr1:~$
```

### 参考

- https://github.com/kubernetes-sigs/kubespray
- https://qiita.com/suzuyui/items/e7531fe5e1e84c061b23
