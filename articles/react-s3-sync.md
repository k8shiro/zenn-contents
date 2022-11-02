---
title: "buildしたReactページをS3にaws-cli s3 syncしようとして失敗した話 --exact-timestampsと--size-onlyのファイルが混ざっていて--deleteもしたい" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["React", "S3", "AWS"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

Reactで作ったページをbuildしてS3にあげて公開しようとしました。

```
/mypage
  ├─data/
  │  ├─****.json
  │  └─****.json
  ├─static/
  │  ├─css/
  │  └─js/
  │  │  └─ main.21292925.jsなど
  ├─index.html
  .
  .
  .
  └─favicon.ico
```

ディレクトリ構成は上記のような作りで、`/mypage/data`にReactページとは別にjsonファイルが置いてあり、それ以外`static/`以下がReactをbuildした時に生成されるファイルです。

これをローカルと同期させようとして

```
aws-cli s3 sync --size-only --delete /mypage s3://******/mypage/ --cache-control max-age=21600 #※ダメな例
```

のようにしたところ公開したところ、Reactページが真っ白になってしまいました。
このコマンドにした意図としては`data/`ディレクトリにあるjsonはファイル名が変わることがあり、また中身の変更がなくてもtimestampsが変更されることがあるので`--size-only`オプションを指定し、使用しなくなったjsonやReactページ内でも不要になったものを削除するように`--delete`をつけていました。


しかし、Reactの場合buildするたびに`static/js/main.*****.js`のファイル名が変わってしまいます。一方でindex.htmlは内部でimportする`static/js/main.*****.js`だけ変わってファイルサイズはそのままになることがあります。このため、`index.html`は更新されずに、`static/js/main.*****.js`だけが更新され(前回のファイルが削除され、ファイル名が変わったものがアップされる)結果的に`index.html`がimportする`static/js/main.*****.js`が存在しない状態になってしまいました。

ということで最終的に以下のようにしました。

```
aws-cli s3 sync --exclude /mypage/data/* --exact-timestamps /mypage s3://******/mypage/ --cache-control max-age=21600
aws-cli s3 sync --size-only --delete /mypage s3://******/mypage/ --cache-control max-age=21600
```

1度目のsyncでdataディレクトリを除いてtimestampの変更を見て更新します。これでindex.htmlは確実に更新されます。また、この時`--delete`をつけていないので前回buildの`static/js/main.*****.js`等は残ってしまっています。

2度目のsyncでdataディレクトリが更新されます。また、--deleteオプションが付いているので前回buildの`static/js/main.*****.js`等は削除されます。



