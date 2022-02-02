---
title: "KubernetesのCronJobのタイムゾーンについて調べてみた(オンプレ・Microk8s・EKS)" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["kubernetes", "aws", "microk8s"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

KubernetesではCronJobマニフェストを作成して、Jobを定期的に実行することができます。
CronJobマニフェストには`schedule: "*/1 * * * *"`のようにいわゆるLinuxのcron と同じ形式でJobの実行タイミングを指定してできますが、この時、タイムゾーンがUTCなのかJSTなのかで実行されるタイミングが変わってしまいます。そこで、CronJobの実行されるタイムゾーンがどこに依存するのか調べました。

# CronJobのタイムゾーン
## 基本的な環境(kubeadm等で構築されたKubernetes環境)

基本的にUTCで動きます。

CronJobのタイムゾーンはkube-controller-managerに依存すると記載されています。

> すべてのCronJobスケジュール: 時刻はジョブが開始されたkube-controller-managerのタイムゾーンに基づいています。
> コントロールプレーンがkube-controller-managerをPodもしくは素のコンテナで実行している場合、CronJobコントローラーのタイムゾーンとして、kube-controller-managerコンテナに設定されたタイムゾーンを使用します。

参考: https://kubernetes.io/ja/docs/concepts/workloads/controllers/cron-jobs/

kubeadm等で作成されたk8s環境では、kube-controller-managerはkube-systemのnamespace内にpodとして起動するので、ホストマシンのタイムゾーンには依存しなく、特別なことをしなければUTCになりそうです。

実際にkubeadmで構築されたホスト側がJSTなk8s環境でCronJobを試したところUTC時刻で動作しました。

## Microk8sの場合

ホスト側と同じタイムゾーン、ホスト側がJSTならCronJobもJSTで動きます。

microk8sの場合kube-controller-manager相当のものがsystemdで動くらしいのでそのためではないかと思います。

参考: https://microk8s.io/docs/configuring-services


## EKSの場合

UTCで動き、UTC以外には変えられないっぽい。。。

参考: https://forums.aws.amazon.com/thread.jspa?threadID=309851

実際、EKS上で確認したところ、kube-controller-managerは起動していないように見えました。

```
$ kubectl get pod --all-namespaces | grep kube-controller-manager
※ 何も出力されない
```

ただし、公式のドキュメント上で明言されている箇所を見つけられなかったので利用時には一度動作確認したほうがよいかもしれません。

# Kubernetes v1.21(1.22 ?)以降で使える`CRON_TZ`環境変数でマニフェストからタイムゾーンを指定する方法(非推奨)

```
spec:
  schedule: "CRON_TZ=Etc/UTC */5 * * * *"
```

のようにマニフェスト側に書ける見たいです。
ただし、英語ドキュメントだと`FEATURE STATE: Kubernetes v1.21 [stable]`となっていて`CRON_TZ`か`TZ`環境変数について記載があり、推奨されないような注意がされています。
参考: https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/
