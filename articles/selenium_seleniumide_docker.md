---
title: "Selenium+Selenium IDEでちゃちゃっとブラウザを自動操作する[Docker・Python]" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["selenium"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

[Selenium](https://www.selenium.dev/ja/documentation/)はプログラムからブラウザを自動操作するためのツールです。Seleniumが用意しているWebDriverを各言語(Python, Ruby, JavaScriptなど)から呼び出し利用します。主にフロントエンドのテストやクローリング・スクレイピングなどに使われます。この記事では、SeleniumをPythonベースで動かす環境をDockerで作成する方法と、Seleniumを動かすPythonプログラムをSelenium IDEをつかって自動生成する方法を解説します。

# Selenium IDEを使って自動操作コードを出力
Selenium IDEとはブラウザの拡張として(Chrome・Firefoxで)提供され、ブラウザ上で行った操作の記録・再生やSeleniumドライバを動かすコードとして出力できるツールです。
Selenium IDEを使って`https://zenn.dev/`を自動操作するコードを出力してみます。手順は以下のgifを参照してください。

- gifの内容
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

Exportしたpythonコードは`selenium_tset.py`という名前に変更して後で使います。


# Seleniumを動かす環境をDockerで作る
以下の図のように2つのDockerコンテナで動作環境を作ります。

![selenium-docker_env.png](/images/selenium-docker_env.png)

必要なものは、`docker-compose.yml`と`Dockerfile`、先ほどExportしたpythonコードの`selenium_tset.py`で以下のように配置していきます。完成したものが[こちらのリポジトリ](https://github.com/k8shiro/selenium-docker-sample_zenn-article)においてあるので参照してください。

```
.
├── docker-compose.yml
└── python
    ├── Dockerfile
    └── selenium_test.py
```

まずはdocker-compose.ymlですが、以下のようになります。

```
version: "3"
services:
  chrome:
    image: selenium/standalone-chrome:4.0.0-rc-1-prerelease-20210804
    shm_size: 2gb
    ports:
      - "4444:4444"
      - "7900:7900"
  python:
    build: ./python
    volumes:
      - ./python:/python
    tty: true
```


まずchromeコンテナですが、[selenium/standalone-chrome:4.0.0-rc-1-prerelease-20210804](https://hub.docker.com/r/selenium/standalone-chrome) を使います。Selenium関連の[公式Docker image](https://hub.docker.com/u/selenium)はいろいろな種類があるのですが、このimageは[docker-seleniumgithubのREADME.md](https://github.com/SeleniumHQ/docker-selenium)で説明されているimageでseleniumのWebDriver、chromeブラウザ、noVNCが入っています。4444ポートはWebDriver用で、pythonコンテナから4444ポート向けにseleniumコードを実行します。7900ポートはnoVNCサーバが立ち上がってます。コンテナ外のブラウザからhttp://<localhost or dockerのホスト>:7900/のようにアクセスするとnoVNCにつながり、seleniumを実行したときにコンテナ内でブラウザが自動祖刺される様子を見ることができます。

そして、pythonコンテナはSelenium IDEで作成した`selenium_tset.py`を実行するためのコンテナです。`selenium_tset.py`が入っている`python/`ディレクトリをマウントしていて、また、落ちないように`tty: true`が設定してあります。Pythonとselenium、pytestが必要なので以下のようにDockerfileを作成します。

```
FROM python:3.9-buster

RUN pip install pip install selenium==4.0.0b4 pytest
```

最後に`selenium_tset.py`を`python/`ディレクトリに配置すれば、以下のように実行できるのですが、先に`selenium_tset.py`を少し修正する必要があります。

```
docker-compose up -d
docker-compose exec python pytest /python/selenium_test.py
docker-compose down
```

# `selenium_tset.py`の修正








