---
title: "Zabbix6.0をk8sにあげるマニフェスト" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["kubernetes", "zabbix"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

Kubernetes上にZabbix 6.0を上げるマニフェストです。
Helmが使える人はHelm使ったほうが良いかもしれません。

- Helm Chart For Zabbix : https://artifacthub.io/packages/helm/aekondratiev/zabbix-server

```
apiVersion: v1
kind: Namespace
metadata:
  name: zabbix

---

apiVersion: v1
kind: Service
metadata:
  name: zabbix-service
  namespace: zabbix
spec:
  selector:
    app: zabbix-service
  type: LoadBalancer
  ports:
   -  name: zabbix-server
      protocol: TCP
      port: 10051
      targetPort: 10051
   -  name: zabbix-web
      protocol: TCP
      port: 8080
      targetPort: 8080
   -  name: zabbix-agent
      protocol: TCP
      port: 10050
      targetPort: 10050

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: zabbix-service
  namespace: zabbix
  labels:
    app: zabbix-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: zabbix-service
  template:
    metadata:
      labels:
        app: zabbix-service
    spec:
      containers:
      - image: zabbix/zabbix-server-pgsql:ubuntu-6.0-latest
        name: zabbix-server
        ports:
        - containerPort: 10051
          protocol: TCP
        env:
        - name: DB_SERVER_HOST
          value: "localhost"
        - name: DB_SERVER_PORT
          value: "5432"
        - name: POSTGRES_USER
          value: "postgres"
        - name: POSTGRES_PASSWORD
          value: "password"
      - image: zabbix/zabbix-web-nginx-pgsql:centos-6.0-latest
        name: zabbix-web-nginx-pgsql
        ports:
          - protocol: TCP
            containerPort: 8080
        env:
        - name: ZBX_SERVER_HOST
          value: "localhost"
        - name: DB_SERVER_HOST
          value: "localhost"
        - name: DB_SERVER_PORT
          value: "5432"
        - name: POSTGRES_USER
          value: "postgres"
        - name: POSTGRES_PASSWORD
          value: "password"
        - name: PHP_TZ
          value: "Asia/Tokyo"
      - image: postgres:13-alpine
        name: zabbix-db
        ports:
        - containerPort: 5432
          protocol: TCP
        env:
        - name: POSTGRES_PASSWORD
          value: "password"
      - image: zabbix/zabbix-agent:6.0-centos-latest
        name: zabbix-agent
        ports:
        - containerPort: 10050
          protocol: TCP
        env:
        - name: ZBX_SERVER_HOST
          value: "localhost"

```

DBは永続化していない状態なので再起動すると消えます。
zabbix-web-nginx-pgsql:centos-6.0-latestをつかっているのはubuntuのイメージを使ったらphpのエラーが表示されていたためです。
