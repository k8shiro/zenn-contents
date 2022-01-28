---
title: "【OS 14種類】Docker imageごとのタイムゾーン設定方法まとめ" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["kubernetes", "docker", "timezone", "linux"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

Dockerでコンテナを立ち上げるとき、dateコマンド等で使われるタイムゾーンを設定する方法をまとめました。

# ホストのタイムゾーンは関係ない
Dockerコンテナの場合はホストのタイムゾーンに関係なく、コンテナ内のOSによって挙動が変わります。

```
# ホスト側(JST)でdateコマンド実行
$ date
2022年  1月 26日 水曜日 11:05:11 JST

# コンテナを立ち上げてdateコマンドを実行
$ docker run alpine date
Wed Jan 26 02:05:20 UTC 2022
```

# Dockerコンテナのタイムゾーンの設定方法

基本的には2種類の方法があります。

- タイムゾーンの設定方法(JSTにする場合)
  - 起動時に環境変数`TZ=Asia/Tokyo`を設定する
  - ホストにある`/usr/share/zoneinfo/Asia/Tokyo`でコンテナ内の`/etc/localtime`を上書き

環境変数を設定する方法が簡単ですが、使用するimageのOSによっては機能しないことがあるので注意が必要です。

**alpineの場合**


alpineの場合`/etc/localtime`も`/usr/share/zoneinfo/`も存在していないため、素のalpineでは環境変数で`TZ`を渡してもタイムゾーンはUTCのままです。
ホストにある`/usr/share/zoneinfo/Asia/Tokyo`をコンテナの`/etc/localtime`にコピーかマウントしてあげるとタイムゾーンを変えることができます。

```
$ docker run alpine date
Wed Jan 26 02:05:20 UTC 2022
$ docker run -e TZ=Asia/Tokyo alpine date
Thu Jan 27 03:22:41 UTC 2022
$ docker run -v /usr/share/zoneinfo/Asia/Tokyo:/etc/localtime alpine date
Thu Jan 27 12:53:00 JST 2022
```

**debianの場合**

stretch/buster/jessie/bullseye等の名前がつくimageはdebianをベースに作られています。
環境変数で`TZ`を渡すとタイムゾーンを変更でき、ホストにある`/usr/share/zoneinfo/Asia/Tokyo`をコンテナの`/etc/localtime`にコピーかマウントしてあげても変更できます。

```
$ docker run debian date
Thu Jan 27 03:20:50 UTC 2022
$ docker run -e TZ=Asia/Tokyo debian date
Thu Jan 27 12:22:10 JST 2022
$ docker run -v /usr/share/zoneinfo/Asia/Tokyo:/etc/localtime debian date
Thu Jan 27 12:55:20 JST 2022
```

# OSごとの挙動の違い

Docker Hub上のofficialイメージでcategory=os、architecture=x86-64(amd64)のイメージを対象に環境変数設定する方法とlocaletimeを上書きをする方法での挙動を調べました。

対象: [official・OS・x86-64のイメージ(2022/01/28時点)](https://hub.docker.com/search?type=image&image_filter=official&category=os&architecture=amd64)

**各方法ごとのdateコマンド実行結果**

| OS | デフォルト | 環境変数設定 | localetime上書き |
| -- | -- | -- | -- |
| alpine | Thu Jan 27 06:23:48 UTC 2022 | Thu Jan 27 06:23:49 UTC 2022 | Thu Jan 27 15:23:50 JST 2022 |
| debian | Thu Jan 27 06:23:51 UTC 2022 | Thu Jan 27 15:23:52 JST 2022 | Thu Jan 27 15:23:53 JST 2022 |
| ubuntu | Thu Jan 27 06:23:54 UTC 2022 | Thu Jan 27 06:23:55 Asia 2022 | Thu Jan 27 15:23:56 JST 2022 |
| amazonlinux | Thu Jan 27 06:23:57 UTC 2022 | Thu Jan 27 15:23:58 JST 2022 | Thu Jan 27 15:23:59 JST 2022 |
| fedora | Thu Jan 27 06:24:00 UTC 2022 | Thu Jan 27 15:24:01 JST 2022 | Thu Jan 27 15:24:02 JST 2022 |
| oraclelinux | Thu Jan 27 06:24:03 UTC 2022 | Thu Jan 27 15:24:04 JST 2022 | Thu Jan 27 15:24:05 JST 2022 |
| ros | Thu Jan 27 06:24:07 UTC 2022 | Thu Jan 27 15:24:08 JST 2022 | Thu Jan 27 15:24:09 JST 2022 |
| photon | Thu Jan 27 06:24:10 UTC 2022 | Thu Jan 27 06:24:11 Asia 2022 | Thu Jan 27 15:24:12 JST 2022 |
| clearlinux | Thu Jan 27 06:24:13 UTC 2022 | Thu Jan 27 06:24:14 Asia 2022 | Thu Jan 27 15:24:15 JST 2022 |
| mageia | Thu Jan 27 06:24:16 UTC 2022 | Thu Jan 27 15:24:17 JST 2022 | Thu Jan 27 15:24:18 JST 2022 |
| sourcemage | Thu Jan 27 06:24:19 UTC 2022 | Thu Jan 27 15:24:21 JST 2022 | Thu Jan 27 15:24:22 JST 2022 |
| crux | Thu Jan 27 06:24:23 UTC 2022 | Thu Jan 27 15:24:24 JST 2022 | Thu Jan 27 15:24:25 JST 2022 |
| cirros | Thu Jan 27 06:24:26 UTC 2022 | Thu Jan 27 06:24:27 UTC 2022 | Thu Jan 27 06:24:28 UTC 2022 |
| centos | Thu Jan 27 06:24:29 UTC 2022 | Thu Jan 27 15:24:30 JST 2022 | Thu Jan 27 15:24:31 JST 2022 |

環境変数設定の場合、タイムゾーンがAsiaとなってしまっているものがありますが、時刻を見るとUTCのままでタイムゾーンの変更に失敗していることがわかります。

**各方法でのタイムゾーン変更の可否**

上記結果をまとめると以下のようにまります。

| OS | 環境変数設定 | localetime上書き |
| -- | -- | -- |
| alpine | × | 〇 |
| debian | 〇 | 〇 |
| ubuntu | × | 〇 |
| amazonlinux | 〇 | 〇 |
| fedora | 〇 | 〇 |
| oraclelinux | 〇 | 〇 |
| ros | 〇 | 〇 |
| photon | × | 〇 |
| clearlinux | × | 〇 |
| mageia | 〇 | 〇 |
| sourcemage | 〇 | 〇 |
| crux | 〇 | 〇 |
| cirros | × | × |
| centos | 〇 | 〇 |

cirrosのみlocaletime上書きの方法でもタイムゾーンを変更できませんでしたが、テスト用のイメージなので実際に困る場面はないのかと思われます。

> CirrOS は、OpenStack Compute のようなクラウドでテストイメージとして使用するために設計された、最小の Linux ディストリビューションです。 CirrOS ダウンロードページ から CirrOS をさまざまな形式でダウンロードできます。
https://docs.openstack.org/ja/image-guide/obtain-images.html

# まとめ

Dockerコンテナのタイムゾーンを変更する場合、localetimeを上書きする方法であれば確実です。しかしKubernetesなどで動かす際にコンテナ内のファイルを上書きする方法は不都合な場合もあります。その場合には、使用しているイメージがどのOSかを確認して環境変数を設定する方法が使えるかを確認しましょう。
