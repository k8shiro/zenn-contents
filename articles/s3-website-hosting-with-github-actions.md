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

- ① バケットの作成をクリックします。

![s3-001](https://storage.googleapis.com/zenn-user-upload/ba60c7a469a0ec336e22e258.png)

- ② バケット名とリージョンを指定します。
  - バケット名は全世界で唯一の名前である必要があります。
  - ここ以外はデフォルトのままでバケットを作成します。

![s3-002](https://storage.googleapis.com/zenn-user-upload/a49aeda6705e31df1a55e870.png)

- 作成したバケットをクリックしてアクセス許可のタブを開いてください。バケットの公開の設定をします。
  - ③ ブロックパブリックアクセスを編集して画像のようにすべてオフにします。
  - ④ バケットポリシーを編集します。
    - ポリシーの内容は以下の通りに入力します。Resource部分は作成したバケットの名前に修正してください。
      - 公式のドキュメントは[ここ](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/WebsiteAccessPermissionsReqd.html)
 
 バケットポリシー
 ```
 {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::＜作成したバケットの名前＞/*"
            ]
        }
    ]
}
 ```

![s3-003-004](https://storage.googleapis.com/zenn-user-upload/9a3204eec8e7f137dd0eaeac.png)

