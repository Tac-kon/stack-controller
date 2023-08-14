# ArgoCD
構築したKubernetesクラスタに対して、[ArgoCD](https://argo-cd.readthedocs.io/en/stable/)の設定を行います。


ArgoCDは主にKubernetesクラスタへのCI/CDパイプラインの構築に使用されます。
ArgoCDを特定のGithubリポジトリに接続すると、リポジトリに更新があった場合に更新が自動でKubernetesクラスタに反映されるようになります。

今回はOpenStackのコントローラーのKubernetesクラスタへのデプロイを、ArgoCDを使用して行えるようにしたいと思います。


## デプロイ
まずはコントローラ1にログインして、`argocd`用のnamespaceを作成します。

```
ubuntu@cr1:~$ kubectl create ns argocd
namespace/argocd created
ubuntu@cr1:~$ kubectl get ns
NAME              STATUS   AGE
argocd            Active   3s
default           Active   157m
kube-node-lease   Active   157m
kube-public       Active   157m
kube-system       Active   157m
```

続いてargocdのデプロイを行います。
ArgoCDのマニフェストファイルには https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml をフォークしたものを使用します。
```
ubuntu@cr1:~$ mkdir argocd
ubuntu@cr1:~$ cd argocd/

ubuntu@cr1:~/argocd$ wget https://github.com/Tac-kon/stack-controller/tree/main/argocd/install.yaml
ubuntu@cr1:~/argocd$ kubectl apply -n argocd -f install.yaml


ubuntu@cr1:~/argocd$ kubectl patch service -n argocd argocd-server --type='json' -p='[{"op": "replace", "path": "/spec/type", "value": "NodePort"}]'
ubuntu@cr1:~/argocd$ kubectl patch service -n argocd argocd-server --type='json' -p='[{"op": "replace", "path": "/spec/ports/1/nodePort", "value": 30443}]'
```

`kubectl get pods -n argocd`を実行して、すべてのpodが起動していたら、デプロイは完了です。
podが起動している状態とは`STATUS`が`Running`になっていることを指します。
```
ubuntu@cr1:~/argocd$ kubectl get pods -n argocd
NAME                                                READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                     1/1     Running   0          9m20s
argocd-applicationset-controller-5787d44dff-q5bz2   1/1     Running   0          9m20s
argocd-dex-server-858cfd495f-5cmnv                  1/1     Running   0          9m20s
argocd-notifications-controller-5d889fdf74-rxl44    1/1     Running   0          9m20s
argocd-redis-7d8d46cc7f-2kdr2                       1/1     Running   0          9m20s
argocd-repo-server-7b6d785784-pvktw                 1/1     Running   0          9m20s
argocd-server-67f667d48c-jj9wc                      1/1     Running   0          9m20s
ubuntu@cr1:~/argocd$
```

## コマンドラインツールのインストール
続いてコントローラ1上で`argocd`コマンドを使用できるようにセットアップします。
下記コマンドを実行してください。
```
ubuntu@cr1:~$ VERSION=2.8.0
ubuntu@cr1:~$ curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v${VERSION}/argocd-linux-amd64
ubuntu@cr1:~$ sudo chmod +x /usr/local/bin/argocd
ubuntu@cr1:~$ echo "argocd completion bash" | sudo tee /etc/bash_completion.d/argocd
```

## ログイン(CLI)
デプロイしたArgoCDにログインしてみましょう。

まずはArgoCDのログインパスワードを確認します。次のコマンドで出力された値がパスワードになります。
```
ubuntu@cr1:~$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d ; echo
```

では次のコマンドを入力してログインしてみましょう。

```
ubuntu@cr1:~$ argocd login 10.0.1.2:30443
```
入力後`username`と`password`について聞かれるので、それぞれ`admin`と先ほど確認したパスワードを入力しましょう。

ログインに成功したら、初期パスワードを任意のものに変更しましょう。次のコマンドでパスワードを変更できます。
```
ubuntu@cr1:~$ argocd account update-password
```

ここまで完了したら、いよいよArgoCDとGithubリポジトリを接続します。
```
ubuntu@cr1:~$ kubectl create ns controller
ubuntu@cr1:~$ argocd app create stack-controller --repo git@github.com:Tac-kon/stack-controller.git --path manifests --dest-server https://kubernetes.default.svc --dest-namespace controller
```

ここで、各オプションについて簡単に説明します。
 - --repo: githubのリモートリポジトリのURLを指定します。
 - --path: 同期するディレクトリを指定します。
 - --dest-server: アプリケーションを動かすKubernetesクラスタを指定します。
 - --dest-namespace: アプリケーションを動かすnamespaceを指定します。

## ログイン(WebUI)
ブラウザを開き、https://10.0.1.2:30443 にアクセスしてみてください。

ログイン情報は
 - ユーザー名: admin
 - パスワード: 先ほど設定したパスワード

です。正しくログインできた場合、次のような画面が表示されるはずです。

![ArgoCD_WebUI](../images/argocd-ui.png)


### 参考
 - https://www.blog.slow-fire.net/2022/05/24/%E4%BB%8A%E6%9B%B4%E3%81%AA%E3%82%89%E3%81%8Cargo-cd%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%BF%E3%81%9F/
 - https://medium.com/penguin-lab/change-clusterip-to-noderport-via-kubectl-patch-command-d2e0279a66cd



name: https
    nodePort: 31414
