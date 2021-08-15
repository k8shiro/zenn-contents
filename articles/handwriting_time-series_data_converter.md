---
title: "機械学習とかで使う時系列データを手書きのグラフからcsvで出力するブラウザ上で動くツールを作った" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["csv", "tool", "canvas", "javascript"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---


# 背景
機械学習等で時系列データを入力とするようなプログラムの開発を行うときに、当然、デバッグやテスト用のデータが必要です。しかしながら、例えば1分間隔で1週間分のデータであれば60*24*7=10080行のデータになり、これを手作業で作るのは大変です。そのため、sin関数等を組み合わせて、それっぽいデータを出力する関数をでっちあげることになるのですが、これはこれでデータ中に異常値を含める必要があったりと、期待したデータを作るのが難しいことがあります。そこで、グラフの外形を手書きして、これをもとに時系列データを生成するツールを作成してみました。

# demo
デモを以下のページで公開しています。
demo: https://k8shiro.github.io/handwriting_time-series_data_converter/

以下のような感じで時系列のデータをcsvファイルで生成できます。
![timeseries-tool.gif](/images/timeseries-tool.gif)

# 使い方
以下のリポジトリのindex.htmlが本体です。index.htmlで完結しているのでローカルにダウンロードしたindex.htmlをHTMLのcanvasが動くブラウザ(Chromeで動作確認)で使用できると思います。細かい使い方は上記のgifかindex.htmlの先頭部分に書いてあります。

リポジトリ: https://github.com/k8shiro/handwriting_time-series_data_converter
本体: https://github.com/k8shiro/handwriting_time-series_data_converter/blob/main/index.html
