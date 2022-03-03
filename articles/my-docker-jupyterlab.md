---
title: "【docker-compose】JupyterLab立ち上げセット" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JupyterLab", "Jupyter Notebook", "docker"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

docker-composeでJupyterLabを立ち上げるセットです。
ちょっとしたpythonのコードやパッケージの動作確認に使います。

- [立ち上げセット](https://github.com/k8shiro/my-jupyter-notebook): https://github.com/k8shiro/my-jupyter-notebook

# 使い方

**起動**

```
docker-compose up -d
```

**接続**

立ち上げたホストへブラウザからポート番号8888を指定して接続してください。  
JupyterLabのログイン画面のパスワードは `docker-compose.yml`のJUPYTER_PASSを変更(default: password)することで設定できます。


**その他**

- hostからコンテナに渡したいデータがある場合は./dataに格納。  
- host側の`./ipynb/`は`/home/jovyan/work`にマウントされている。
- `./data`と`./ipynb`は適切にpermissionをつけてください。
- pythonのパッケージを追加したいときは起動前に`jupyter/requirements.txt`に追加。
- コンテナ内にgitをインストール済みなのでJupyterLabのターミナルからgitコマンドが使えます。
