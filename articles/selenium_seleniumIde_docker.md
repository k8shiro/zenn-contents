---
title: "Selenium+Selenium IDEでちゃちゃっとブラウザを自動操作する[Docker・Python]" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["selenium"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

[Selenium](https://www.selenium.dev/ja/documentation/)はプログラムからブラウザを自動操作するためのツールです。Seleniumが用意しているWebDriverを各言語(Python, Ruby, JavaScriptなど)から呼び出し利用します。主にフロントエンドのテストやクローリング・スクレイピングなどに使われます。この記事では、SeleniumをPythonベースで動かす環境をDockerで作成する方法と、Seleniumを動かすPythonプログラムをSelenium IDEをつかって自動生成する方法を解説します。

# Selenium IDEを使う
Selenium IDEとはブラウザの拡張として(Chrome・Firefoxで)提供され、ブラウザ上で行った操作の記録・再生やSeleniumドライバを動かすコードとして出力できるツールです。
Selenium IDEを使って`https://zenn.dev/`を自動操作するコードを出力してみます。手順は以下のgifを参照してください。

- Selenium IDE起動
- Create a new project(プロジェクトの作成)
- テストの作成
- レコーディングの開始
- base URL（自動操作を始めたいURL）の入力
  - `https://zenn.dev/` を入力
- ブラウザを自動操作
- レコーディング停止
- レコーディング内容の確認
- Export
  - PythonでExport
- Save Project 

![selenium_zenn2.gif](/images/selenium_zenn2.gif)
