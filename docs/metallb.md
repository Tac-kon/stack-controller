# MetalLB

一般的に Kubernetes 上にデプロイしたアプリケーションを外部公開する際は`Service`リソースを使用するケースが多いです。

そして、今回はその Service リソースの一種である`LoadBalancer`を使用します。LoadBalancer を使用することで Service のエンドポイントとなる IP アドレスを払いだすことができます。

しかしながら、一般的に LoadBalancer はクラウド環境下での使用が想定されており、オンプレミス環境で使用するためには[MetalLB](https://metallb.universe.tf/)などの導入が必要になります。
今回はその MetalLB の適用方法ご紹介します。

## 適用

早速 [Installation](https://metallb.universe.tf/installation/) の手順を参考に、MetalLB を適用します。

```
# see what changes would be made, returns nonzero returncode if different
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system

# actually apply the changes, returns nonzero returncode on errors only
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system

METALLB_VERSION=0.13.12
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v${METALLB_VERSION}/config/manifests/metallb-native.yaml
```

次に、MetalLB の設定を記述した config ファイルを適用します。
config ファイルは[metallb/config.yaml](../metallb/config.yaml)のものを使用します。

config ファイル内の下記の部分の、Service に割り当てる IP アドレスの範囲を環境に応じて適宜変更してください。

```
addresses:
      - 10.0.1.201-10.0.1.240
```

IP アドレスの範囲調整後、config ファイルを適用します。

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

この後に、`lb-sample`の`EXERNAL-IP`が設定されていれば OK です。

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
