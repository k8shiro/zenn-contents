---
title: "【Python】ディレクトリトラバーサル攻撃と対策" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# ディレクトリトラバーサル

> ディレクトリトラバーサル (英語: directory traversal) とは、利用者が供給した入力ファイル名のセキュリティ検証/無害化が不十分であるため、ファイルAPIに対して「親ディレクトリへの横断 (traverse)」を示すような文字がすり抜けて渡されてしまうような攻撃手法のことである。

wikipediaより: https://ja.wikipedia.org/wiki/%E3%83%87%E3%82%A3%E3%83%AC%E3%82%AF%E3%83%88%E3%83%AA%E3%83%88%E3%83%A9%E3%83%90%E3%83%BC%E3%82%B5%E3%83%AB# 

# 駄目な例

flaskを使ったWebサーバーの例です。nameとmessageをPOSTすると`/src/public/`にファイルが保存され、GETで指定されたnameのmessageを取得できるという単純なコードです。

```
from flask import Flask, request
app = Flask(__name__, static_folder='.', static_url_path='')

@app.route('/', methods=['GET', 'POST'])
def index():
  if request.method == 'GET':
    """
    curl http://127.0.0.1:8000/?name=20220101.txt
    """
    name = request.args.get('name', '')
    file = '/src/public/' + name
    with open(file) as f:
        text = f.read()
        return text
  elif request.method == 'POST':
    """
    curl http://127.0.0.1:8000/ -X POST \
      -H "Content-Type: application/json" \
      --data '{"name": "20220101.txt", "message": "メッセージ\n"}'
    """
    name = request.json['name']
    message = request.json['message']

    file = '/src/public/' + name
    with open(file, 'w') as f:
      f.write(message)

    return 'save\n'

app.run(port=8000, host='0.0.0.0', debug=True)
```

このコードでは、

```
    curl http://127.0.0.1:8000/ -X POST \
      -H "Content-Type: application/json" \
      --data '{"name": "20220101.txt", "message": "メッセージ\n"}'
```

のようにPOSTした後、

```
curl http://127.0.0.1:8000/?name=20220101.txt
```

のようにGETを投げると`20220101.txt`の中に保存されているmessageを見ることができます。  
ここで、'/src/public/'ではなく'/src/private/secret.txt'という公開していないつもりのファイルがあったとします。  
このファイルはGETリクエストで

```
curl http://127.0.0.1:8000/?name=../private/secret.txt
```

のようにnameに`../`のような親ディレクトリの参照を含ませることでprivateディレクトリの中にあるファイルが外部から参照できてしまいます。


# 安全な例① ハッシュ化する

ユーザーの入力が直接ファイル名になっているのが問題なので、保存時にファイル名をハッシュ化することでディレクトリトラバーサルの対策になります。  
保存と読み込みの両方で`/src/public/`とnameを連結する前にnameをハッシュ化することで'../'等の問題のある文字列が含まれることを防いでいます。


```
from flask import Flask, request
import hashlib
app = Flask(__name__, static_folder='.', static_url_path='')

@app.route('/', methods=['GET', 'POST'])
def index():
  if request.method == 'GET':
    """
    curl http://127.0.0.1:8000/?name=20220101.txt
    """
    name = request.args.get('name', '')
    hash_name = hashlib.md5(name.encode()).hexdigest() # ハッシュ化
    file = '/src/public/' + hash_name                  # ハッシュ化して連結
    with open(file) as f:
        text = f.read()
        return text
  elif request.method == 'POST':
    """
    curl http://127.0.0.1:8000/ -X POST \
      -H "Content-Type: application/json" \
      --data '{"name": "20220101.txt", "message": "メッセージ\n"}'
    """
    name = request.json['name']
    message = request.json['message']

    hash_name = hashlib.md5(name.encode()).hexdigest()　# ハッシュ化
    file = '/src/public/' + hash_name                   # ハッシュ化して連結
    with open(file, 'w') as f:
      f.write(message)

    return 'save\n'

app.run(port=8000, host='0.0.0.0', debug=True)
```


# 安全な例② 正規化

`open`する前にパスを正規化してパス名の先頭の共通部分が公開ディレクトリかを確認します。

```
from flask import Flask, request
import os
app = Flask(__name__, static_folder='.', static_url_path='')

@app.route('/', methods=['GET', 'POST'])
def index():
  if request.method == 'GET':
    """
    curl http://127.0.0.1:8000/?name=20220101.txt
    """
    name = request.args.get('name', '')
    file = '/src/public/' + name
    if os.path.commonprefix((os.path.realpath(file),'/src/public/')) != '/src/public/':
      return 'failed'
    with open(file) as f:
        text = f.read()
        return text
  elif request.method == 'POST':
    """
    curl http://127.0.0.1:8000/ -X POST \
      -H "Content-Type: application/json" \
      --data '{"name": "20220101.txt", "message": "メッセージ\n"}'
    """
    name = request.json['name']
    message = request.json['message']

    file = '/src/public/' + name
    if os.path.commonprefix((os.path.realpath(file),'/src/public/')) != '/src/public/':
      return 'failed'
    with open(file, 'w') as f:
      f.write(message)

    return 'save\n'

app.run(port=8000, host='0.0.0.0', debug=True)
```
