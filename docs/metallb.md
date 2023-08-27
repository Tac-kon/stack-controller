# MetalLB
一般的にKubernetes上にデプロイしたアプリケーションを外部公開する際は`Service`リソースを使用するケースが多いです。

そして、今回はそのServiceリソースの一種である`LoadBalancer`を使用します。LoadBalancerを使用することでServiceのエンドポイントとなるIPアドレスを払いだすことができます。

しかしながら、一般的にLoadBalancerはクラウド環境下での使用が想定されており、オンプレミス環境で使用するためには[MetalLB](https://metallb.universe.tf/)などの導入が必要になります。
今回はそのMetalLBの適用方法ご紹介します。

## 適用
早速MetalLBを適用していきましょう。下記のようにマニフェストファイルを適用します。
```
METALLB_VERSION=0.12.1
kubectl apply -f https://raw.githubusercontent.com/google/metallb/v${METALLB_VERSION}/manifests/metallb.yaml
```

次に、MetalLBの設定を記述したconfigファイルを適用します。
configファイルは[metallb/config.yaml](../metallb/config.yaml)のものを使用します。

configファイル内の下記の部分の、Serviceに割り当てるIPアドレスの範囲を環境に応じて適宜変更してください。
```
addresses:
      - 10.0.1.201-10.0.1.240
```

IPアドレスの範囲調整後、configファイルを適用します。
```
kubectl apply -f metallb/config.yaml 
```

## 動作確認
動作確認のために下記のサービスを作成します。
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx-sample
  labels:
    app: nginx-sample
spec:
  containers:
    - image: nginx
      name: nginx-sample
      ports:
      - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: lb-sample
spec:
  ports:
  - port: 80
  selector:
    app: nginx-sample
  type: LoadBalancer
EOF
```

この後に、`lb-sample`の`EXERNAL-IP`が設定されていればOKです。
```
kubectl get svc lb-sample
NAME        TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
lb-sample   LoadBalancer   10.233.50.12   10.0.1.202    80:31210/TCP   9s

curl 10.0.1.202
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
・・・
```

### 参考
 - https://ryusa.hatenablog.com/entry/2021/05/21/231844