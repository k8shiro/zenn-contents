---
title: "Notionでリンク先のテーブルの値を使って計算する" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["notion"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

notionにはdatabaseと呼ばれる機能があります。名前の通りいわゆるデータベースで、これの入出力を行うためのviewが数パターン用意されていて、CalendarやBoard、Tableなどいくつかあります。今回はTableを使って作業していきます。

Tableはexcelっぽい表形式のviewです。  
まずはtableを二つ作ります。ドキュメント上で`/table`と入力して作れます。
![table-command](https://storage.googleapis.com/zenn-user-upload/3fb60bb9950b25c48f634cc1.png)
