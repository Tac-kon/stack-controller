# Rook-Cephのデプロイ
## Rook-Cephとは？
「Rook」と「Ceph」のそれぞれについて、簡単に説明します。

Rookとは、Kubernetes上で動作する分散ストレージの「Operator」を指します。
Rookを使用することで分散ストレージシステムをKubernetes上にデプロイしたり、デプロイしたストレージシステムの構成管理などが可能になります。

続いて[Ceph](https://ceph.io/)はオープンソースの分散ストレージシステムになります。細かい説明は省きますが、Cephは複数のノードを使用してオブジェクトストレージのクラスタを構築し、またクライアントがストレージシステムにアクセスするためのインターフェスを提供します。

つまり、Rook-CephとはKubernetes上でCephをデプロイしたり、構成管理するためのOperatorのことを指します。

Rook-Cephを導入することで、Kubernetes上のリソースが分散ストレージシステムを使用することができます。
今回はKubernetes上で動作するMySQLやCinderなどのリソースのデータを保存する目的でRook-Cephを構築します。

## 構築手順
[公式の手順](https://rook.github.io/docs/rook/v1.12/Getting-Started/quickstart/)を参考に進めていきます。

### Rookリポジトリの取得
まずは作業用PCにRookのリポジトリを取得します。
```
cd ~
ROOK_VERSION=1.12.7
git clone --single-branch --branch v${ROOK_VERSION} https://github.com/rook/rook.git
```

続いて~/rook/deploy/example/cluster.yamlの``placement``以下の"#"を外して、nodeAffinityの設定をコメントインさせます。
これでCephのosd(データの読み書きの際に使用されるデーモン)のpodが``storage-node``のラベルが付与されたノードにのみ展開されるようになります。
```
  placement:
    all:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: role
              operator: In
              values:
              - storage-node
```

### ノード設定
今回はストレージノードにosdを展開するため、下記コマンドでラベルを付与します。
```
STORAGE_NODES="st1 st2 st3"
for NODE in ${STORAGE_NODES}
do
  kubectl label node ${NODE} role=storage-node
done
```

### デプロイ
[deploy-the-rook-operator](https://rook.io/docs/rook/v1.12/Getting-Started/quickstart/#deploy-the-rook-operator)の手順でオペレータをデプロイしてCephクラスタを構築します。

まずは下記コマンドを実行します。
```
cd rook/deploy/examples
kubectl create -f crds.yaml -f common.yaml -f operator.yaml
```

``watch kubectl get pods -n rook-ceph``コマンドを実行してオペレータのデプロイが完了するのを待ちます。
``rook-ceph-operator-xxxxx-xxxxx``のpodがのSTATUSがRunningになればオペレータのデプロイは完了です。

続けて下記コマンドでCephクラスタを構築します。

```
kubectl create -f cluster.yaml
```

再度``watch kubectl get pods -n rook-ceph``コマンドを実行してosdが起動するのを待ちます。
``rook-ceph-osd-x-xxxxx-xxxxx``のpodのSTATUSがRunningになればosdの起動が完了しています。

以上で構築は完了です。


## 動作確認
上記で構築したCephクラスタの動作を確認します。
まずはStorageClassの定義をします。

```
cd ~/rook/
kubectl create -f deploy/examples/csi/rbd/storageclass.yaml
```

``rook-ceph-block``というStorageClassができていれば問題ないです。
```
kubectl get sc
NAME              PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com   Delete          Immediate           true                   5s
```

続いて動作確認用のpodをデプロイします。
rookのリポジトリのサンプルを使用してMySQL podをデプロイします。
```
cd ~/rook/deploy/examples/
kubectl create -f mysql.yaml 
```

ここで、MySQL pod用のボリュームが作成できていれば問題ないです。
具体的にはpvcリソースのSTATUSがBoundになっていることを確認できればOKです。
```
kubectl get pvc
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
mysql-pv-claim   Bound    pvc-df03d83f-a114-401b-8fcd-41946b25517b   20Gi       RWO            rook-ceph-block   16s
```

### 参考
 - https://rook.io/