---
title: "S3に静的サイトを公開してGithub Actionsで更新する" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws", "s3", "github", "github actions"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# 手動でS3に静的サイトを公開する

手動でS3を使用した静的サイトの公開手順はAWSのドキュメントがあります。

- https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/WebsiteHosting.html

## バケットを作成する
まずはS3にバケットを作っておきます。 バケットとは→https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/UsingBucket.html

S3コンソールで作業を行います：https://console.aws.amazon.com/s3/

![s3-001](https://storage.googleapis.com/zenn-user-upload/ba60c7a469a0ec336e22e258.png)


