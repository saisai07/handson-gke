# GKE Hands On
## CloudShellを起動
コンソール画面右上のアイコンからCloudShellを実行

## 1.必要となるAPIの有効化

```
gcloud services enable \
  cloudapis.googleapis.com \
  container.googleapis.com \
  containerregistry.googleapis.com \
  cloudbuild.googleapis.com
```

## 2.クラスター
### 2.1 作成
ノードは最小限の1台にしています

```
gcloud container clusters create gcpug-sendai --enable-ip-alias --num-nodes=1 --zone=us-central1-a --async
```

出来上がるまで時間がかかります

作成確認
```
gcloud container clusters list
```

### 2.2 クラスターに接続
```
gcloud container clusters get-credentials gcpug-sendai --zone us-central1-a
```

ノードも出来ている
```
kubectl get nodes
```

## 3.Namespaceの作成
Namespaceを設定・指定することで、クラスタ内でPodやServiceなどをグルーピングできるようになります

```
kubectl create namespace gcpug
kubectl get namespaces
```

## 4.ビルド

```
cd goapp
```

Cloud Buildを使用してビルドを行います。
```
gcloud builds submit --tag=gcr.io/$PROJECT_ID/gcpug-go:v1 .
```

作成確認
```
gcloud container images list
```

## 5.デプロイ
GKEにコンテナをデプロイします
```
cd goapp
```

```
kubectl apply -f manifests/deployment.yaml
```

Podの確認
```
kubectl -n gcpug get deployments
```
```
kubectl -n gcpug get pods
```

## 6. 公開
### 6.1 Serviceの作成
ServiceとはPodのエンドポイントを一つにグルーピングしたもの
LBにもできるのでまずはLBで作ってみます

```
kubectl apply -f manifests/service.yaml
```

```
kubectl -n gcpug get svc
```

グローバルIPが振られるのでブラウザでアクセス
これで公開が完了

### 6.2 Ingress経由でアクセスするように変更する
service.yamlのtypeをNodePortに変更
```
kind: Service
apiVersion: v1
metadata:
  name: go-app-service
  namespace: gcpug
spec:
  type: NodePort
  selector:
    app: go-app
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
```

変更を適用

```
kubectl apply -f manifests/service.yaml
```

```
kubectl -n gcpug get svc
```

### 6.3 静的IPの確保
LBに割り当てるグローバルIPを取得します

```
gcloud compute addresses create gcpug-go-ip \
     --global \
    --ip-version IPV4
```

### 6.4 Ingressの作成

```
kubectl apply -f manifests/ingress.yaml
```

```
kubectl -n gcpug get ing
```

### 7 おまけ
Pod内に2個コンテナがあるパターン
今回はphpとnginxの２コンテナの構成にしています

```
cd phpapp/php
```

PHPコンテナのビルドを行います。
```
gcloud builds submit --tag=gcr.io/$PROJECT_ID/gcpug-php:v1 .
```

```
cd ../nginx
```

Nginxコンテナのビルドを行います。
```
gcloud builds submit --tag=gcr.io/$PROJECT_ID/gcpug-nginx:v1 .
```

デプロイ

```
cd ../
```

```
kubectl apply -f manifests/deployment.yaml
```


公開
```
kubectl apply -f manifests/service.yaml
```
```
kubectl apply -f manifests/ingress.yaml
```

## お掃除

```
kubectl delete -f goapp/manifests/ingress.yaml
kubectl delete -f goapp/manifests/service.yaml
kubectl delete -f phpapp/manifests/service.yaml
gcloud container clusters delete gcpug-sendai --zone=us-central1-a --async
gcloud compute addresses delete gcpug-go-ip --global
```