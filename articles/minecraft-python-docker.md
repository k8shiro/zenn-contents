---
title: "minecraftサーバーとJupyterLabをDockerで立ち上げてpythonからminecraftする環境を作る" # 記事のタイトル
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
    └── raspberryjuice-1.12.1.jar
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

minecraft-serverはMinecraftサーバーが立ち上がります。サーバーにはSpigotを使用しています（ドキュメントを見る限りほかのModサーバーも選択できそう）。Minecraftのバージョンは`1.8`で立ち上がるのでサーバーにクライアント側から接続する場合は注意してください。古いバージョンを使っているのはpythonから操作するできるようにModの[RaspberryJuice](https://www.spigotmc.org/resources/raspberryjuice.22724/)が新しいMinecraftのバージョンに対応していないためです。RaspberryJuiceのドキュメント上では1.7～1.12.1まで対応しているようなのでもう少し新しいバージョンのMinecraftでも大丈夫かもしれません。

jupyterはJupyterLabが立ち上がります。ブラウザでdockerを動かしているホストの8888ポートにアクセスすればJupyterLabの画面が見えます。アクセスするとパスワードを求められるので`password`と入力してください。

## jupyterディレクトリ
### Dockerfile
jupyterlabを立ち上げるためにDockerfileです。

```
FROM jupyter/datascience-notebook
COPY requirements.txt /home/jovyan/work/requirements.txt
RUN pip install -r /home/jovyan/work/requirements.txt
```

### requirements.txt
py3minepiというRaspberryJuice Modとpythonが通信するためのパッケージをinstallします。

```
git+https://github.com/py3minepi/py3minepi
```


## pluginsディレクトリ
Modをこのディレクトリに入れます。今回必要なModのRaspberryJuice(raspberryjuice-1.12.1.jar)を

- https://www.spigotmc.org/resources/raspberryjuice.22724/

からダウンロードして配置してください。

# 起動
docker-composeでビルドと起動をします。

```
$ docker-compose build
$ docker-compose up -d
```

# 動作確認

JupyterLabでpythonのnotebookを作って以下のコードを実行してください。

```python
from mcpi.minecraft import Minecraft
mc = Minecraft.create("minecraft-server", 4711)
mc.postToChat("Hello Minecraft!!!")
```

`docker-compose logs`で`Hello Minecraft!!!`と出力されれば成功です。

```
$ docker-compose logs -f minecraft-server
～略～
mc-pi               | [10:54:58 INFO]: Hello Minecraft!!!
```

Minecraftサーバーにアクセスしていればチャットにも表示されます。

![hello-minecraft.png](/images/hello-minecraft.png)
