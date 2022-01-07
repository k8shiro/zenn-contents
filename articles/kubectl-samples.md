---
title: "kubectl コマンド samples" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["kubernetes"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---


# 結果をjsonで出力 (-o json)

kubectlの出力をjson形式で取得する場合は`-o json`をつける。
kubectlで何かをするときには`-o jsonpath=~~~`で検索するときが多いのでjsonpathを調べるときに使う。

```
# pod一覧をjson形式で取得
$ kubectl get pods -o json
```

# Podの名前部分だけを取得する。specでつけた名前から検索する。

deploymentなどから立ち上げたときに自動生成される`my-pod-name-cd8c499d-fb86t`のようなpodの名前を取得


```
# my-pod-nameとつけたpodの名前を取得。
$ kubectl get pods -o jsonpath='{.items[?(@.spec.containers[*].name=="my-pod-name")].metadata.name}'
```
