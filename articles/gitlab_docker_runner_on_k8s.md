---
title: "k8s上にGitlab Runnerを立ち上げてdocker buildさせる" # 記事のタイトル
emoji: ":bear:" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

k8s上にdockerコマンドを実行できるGitlab Runnerをデプロイする手順を紹介します。

# 前提
この記事では以下が準備されていることを前提として進めます。

- 構築済みのk8s
- k8sにkubectlコマンドを実行できる開発マシン

この後の作業は基本的にはこの開発マシン上で行っていきます。

# k8s上にnamespace準備
k8s上にrunnerが使うnamespace `gitlab`を作成します。

```
kubectl create namespace gitlab
```

# Gitlab上からRunner Token取得
ブラウザでRunnerを登録したいGitlab上のプロジェクト or グループのページに移動し`Settings` > `CI/CD` の `Runners` セクションを展開してください。画像の赤線部分に表示されています。

![](https://storage.googleapis.com/zenn-user-upload/3chwmewkftpiyjoae2tpuo69jqtc)

# helmのinstall

GitLab Runnerのデプロイにhelmを使用するため、先にhelmコマンドを使えるようにします。インストール方法は「 [helmのinstall](https://helm.sh/ja/docs/intro/install/)」を参照してください。
スクリプトからインストールする場合は以下になります。

```
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```


# helmを使ってGitLab Runnerを作成する
https://docs.gitlab.com/runner/install/kubernetes.html#installing-gitlab-runner-using-the-helm-chart
を参照して helmを使ってGitLab Runnerを作成します。

helmのバージョンを確認しておきます。3以上なら問題なしです。

```
$ helm version
version.BuildInfo{Version:"v3.5.4", GitCommit:"1b5edb69df3d3a08df77c9902dc17af864ff05d1", GitTreeState:"clean", GoVersion:"go1.15.11"}
```

helmのGitlabリポジトリを追加します。

```
helm repo add gitlab https://charts.gitlab.io
```



# Gitlab Runner用のconfig values fileを作る

`gitlab_runner_helm_config.yml`として以下を作ります。これは、この後、helmを使ってGitLab Runnerを作成する時に使います。

```
helm show values gitlab/gitlab-runner > gitlab_runner_helm_config.yml
```

gitlab_runner_helm_config.ymlの内容を変更します。以下は変更箇所だけ書いてあります。

```
replicas: 3
gitlabUrl: https://gitlab.com/
runnerRegistrationToken: "*****"
rbac:
  create: true
runners:
  config: |
    [[runners]]
      [runners.kubernetes]
        image = "ubuntu:20.04"
        privileged = true
        pre_build_script = "until docker info; do sleep 1; done"
      [[runners.kubernetes.volumes.empty_dir]]
        name = "docker-certs"
        mount_path = "/certs/client"
        medium = "Memory"
  tags: "docker"
  name: "runner-on-k8s"
  env:
    DOCKER_HOST: tcp://docker:2376
    DOCKER_TLS_CERTDIR: "/certs"
    DOCKER_TLS_VERIFY: 1
    DOCKER_CERT_PATH: "/certs/client"
```

gitlab_runner_helm_config.ymlを見ると各項目の解説がコメントされているので参照ください。上記以外にもPodのリソースの制限等様々な設定可能項目があります。また、runnersの設定項目はdocs.gitlab.comの[Docker-in-Docker with TLS enabled](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#docker-in-docker-with-tls-enabled)に従って設定しています。ドキュメント上でenvはgitlab-ci.yml上で指定するようにしていますが、このタイミングで設定してしまっています。また`pre_build_script`を追加しています。これはCI実行時にdockerが起動前にpiplineが始まってしまうことがあるためdockerコマンドが使えるようになるまで松処理を入れています。


# Runner 起動
編集後、上記ファイルを指定してhelm installを実行します。

```
helm install --namespace gitlab gitlab-runner -f gitlab_runner_helm_config.yml gitlab/gitlab-runner
```

実行後、少し待ってから以下のようになれば完了です。

```
$ kubectl get all -n gitlab 
NAME                                                     READY   STATUS    RESTARTS   AGE
pod/gitlab-runner-gitlab-runner-7777777777-gb5gq         1/1     Running   0          9m26s
pod/gitlab-runner-gitlab-runner-7777777777-gvb6q         1/1     Running   0          9m26s
pod/gitlab-runner-gitlab-runner-7777777777-v9mrd         1/1     Running   0          9m26s
pod/runner-55555555-project-55555555-concurrent-555555   3/3     Running   0          52s

NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/gitlab-runner-gitlab-runner   3/3     3            3           9m26s

NAME                                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/gitlab-runner-gitlab-runner-7777777777   3         3         3       9m26s
```

# 確認
最後に、Gitlab上にrunnerが登録されているか確認します。



- 参考
    - [helmのinstall](https://helm.sh/ja/docs/intro/install/)
    - [Docker-in-Docker with TLS enabled in Kubernetes](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#docker-in-docker-with-tls-enabled-in-kubernetes)


