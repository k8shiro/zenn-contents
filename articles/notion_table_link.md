---
title: "Notionでリンク先のテーブルの値を使って計算する" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["notion"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

NotionでTableから別Tableの値を参照して合計や平均を出す方法を紹介します。

Notionにはdatabaseと呼ばれる機能があります。名前の通りいわゆるデータベースで、これの入出力を行うためのviewが数パターン用意されていて、CalendarやBoard、Tableなどいくつかあります。今回はTableを使って作業していきます。

# ゴール
以下のようにtable-mainからtable-subが参照され、table-mainでは各行でtable-subの行が複数参照されval1のSUMとval2のAverageが計算されます。
![goal](https://storage.googleapis.com/zenn-user-upload/ab870f597ed47f57a61a94b0.png)

# 作業
Tableはexcelっぽい表形式のviewです。  
まずはTableを二つ作ります。ドキュメント上で`/table`と入力して作れます。

![table-command](https://storage.googleapis.com/zenn-user-upload/3fb60bb9950b25c48f634cc1.png)

## table-sub
table-subはName列・val1列・val2列を作りval1・val2はProprty TypeをNumberにします。

![table-sub](https://storage.googleapis.com/zenn-user-upload/56ed0f4b448359f1f018114b.png)

できたら適当に値を入力してください。

## table-main

table-mainはまずtable-sub列にrelationを設定します。

![relation](https://storage.googleapis.com/zenn-user-upload/b0dc8410e29bfbc7fb88de74.png)

次にval1-sum列にrollupを設定します。
![rollup1](https://storage.googleapis.com/zenn-user-upload/97918772eaa8ef0497a46c05.png)

Configure rollupを選択し、
![](https://storage.googleapis.com/zenn-user-upload/785cabaf7eab6410666ce556.png)

以下のように各項目を設定します。
- RELATIONをtable-sub
- PROPERTYをval1
- CALCULATEをSum

![](https://storage.googleapis.com/zenn-user-upload/7639f5dc44ed3295ec25ce64.png)

val2-avgも同様にCALCULATEをAverageに設定すれば完成です。




