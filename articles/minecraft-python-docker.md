---
title: "minecraftサーバーとJupyterLabをDockerで立ち上げてminecraftをpythonから操作する環境をDockerで立ち上げる" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["minecraft", "python", "docker", "jupyterlab", "jupyternotebook"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

サンドボックスゲームの「Minecraft」はModを追加することでプログラムから操作することができます。今回はプログラムから操作可能な状態のMinecraftサーバーとPython実行用のJupyterLabサーバーをdocker-compsoeでまとめて立ち上げる方法を解説します。


# 作るもの
以下のような構成になるようにファイル等を用意していきます。

```
.
├── docker-compose.yml
├── jupyter
│   ├── Dockerfile
│   └── requirements.txt
└── plugins
    └── raspberryjuice-1.8.jar
```

基本は[こちら](https://github.com/itzg/docker-minecraft-server/wiki/Minecraft-Pi)のドキュメントをベースに手順を作成していますが、ドキュメントどおりに行うと上手く動かなかったので一部修正しています。

## docker-compose.yml
docker-compose.ymlは以下のように作成してください。

```
version: '3'
services:
  minecraft-server:
    image: "itzg/minecraft-server:multiarch"
    container_name: "mc-pi"

    ports:
      #- "25575:25575"
      - "25565:25565"
      - "4711:4711"

    volumes:
      - "./data:/data"
      - "./plugins:/plugins"

    environment:
      EULA: "TRUE"
      TYPE: "SPIGOT"
      VERSION: "1.8-R0.1-SNAPSHOT-latest"

    tty: true
    stdin_open: true
  jupyter:
    build:
      context: ./jupyter
    environment:
      JUPYTER_ENABLE_LAB: 1
    ports:
      - 8888:8888
    volumes:
      - ./jupyter:/home/jovyan/work
    command: start-notebook.sh --NotebookApp.token='password'
```

minecraft-serverはSpigotを使用しています（ドキュメントを見る限りほかのModサーバーも選択できそう）。Minecraftのバージョンは1.8で立ち上がるのでサーバーにクライアント側から接続する場合は注意してください。古いバージョンを使っているのはpythonから操作するできるようにModの[RaspberryJuice](https://www.spigotmc.org/resources/raspberryjuice.22724/)が新しいMinecraftのバージョンに対応していないためです。RaspberryJuiceのドキュメント上では1.7～1.12.1まで対応しているようなのでもう少し新しいバージョンのMinecraftでも大丈夫かもしれません。

