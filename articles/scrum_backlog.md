---
title: "スクラム開発のバックログをどう管理するか考える" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["scrum"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# バックログ

バックログはスクラム開発における作成物で、[スクラムガイド](https://scrumguides.org/docs/scrumguide/v2020/2020-Scrum-Guide-Japanese.pdf)の"スクラムの作成物"に記載があります。
バックログにはプロダクトバックログとスプリントバックログの2種類があり、ここではこの二つをどう管理すべきか考えます。

# バックログとバックログアイテム

バックログはプロダクトの改善に必要なものの⼀覧であり、必要なもの一つ一つをバックログアイテムと呼びます。バックログアイテムはPBIなどと略されます。

## バックログアイテムの特性(INVEST)

良いバックログアイテムが守るべき原則としてINVESTというものがあります。 参考: https://ja.wikipedia.org/wiki/INVEST

- INVEST
  - Independent	独立していること。
    - 依存関係があるとバックログの優先順位を変える時に複雑になるので独立していることが望ましい。
  - Negotiable 交渉可能であること。
    - 例えば、"デザインを見やすくすること"は交渉可能であるが"タイトルの文字サイズを27ptにする"は交渉可能でない。
    - 上記の例の場合、27ptにするために他のデザインが崩れ、それを直すために実装に時間がかかるような事態は望ましくないので交渉可能である必要がある。
  - Valuable 価値があること。	
    - バックログアイテムが完了すればプロダクトの価値(ビジネス価値 or ユーザーにとっての価値)が上がるようになっていること。
  - Estimatable	見積もり可能であること。
    - 見積もりは開発者が行うため、開発者が理解可能なように具体的・明快に記載されている必要がある。
  - Small 十分に小さいこと。
    - 1スプリントで完了できるサイズが最大。
    - なるべく小さい方が望ましい。	
  - Testable テスト可能である。
    - 開発の観点で言えばテスト駆動開発が実施できること。
    - 客観的にバックログアイテムが完了したことを判断できるようにする。

**IndependentとSmall**

バックログアイテムを小さくしていくと、どうしてもAを実行してからBを行うというように依存関係が出てきてしまいます。しかし、A+Bを一つのバックログアイテムにすると大きくなってしまうのでIndependentとSmallの両立が難しくなります。これを解消するためにEpicという考え方を使います。依存関係のある複数のバックログアイテムをEpicとしてバックログに登録し、優先順位を管理します。そして、プロダクトバックログからスプリントバックログへ移す時にEpicをバックログアイテムに分解して扱うようにします。

**EstimatableとNegotiable**

見積もり可能であることは必要なことですが、スクラム開発の見積もりはあくまで目安程度のものとして扱うべきです。見積もりの精度も、優先度の低いうちは曖昧で問題なく、プロダクトバックログからスプリントバックログに移動する時に詳細が分かるようになるのが望ましいです。。また、Negotiableを侵害するほどの詳細な設計は不要であり、

- Who 誰のためのものか
- What 何をしたいのか
- Why なぜそうしたいのか

をはっきりさせることが望ましいです。

**ValuableとTestable**

バックログアイテムが価値があるかは最終的にプロダクトオーナーが判断して登録することになるでしょう(なぜValuableであるか説明されている必要がある)。そしてTestableであるということは"価値がある"状態になったことを確認できることです。



## バックログアイテムの種類

スクラムで開発者が行う作業は全てバックログアイテムとして登録すべきだと考えています。しかし、技術的負債の解消のようなエンドユーザーにとって直接的には価値のないものもあるため

- ユーザーストーリー
- バグ修正
  - バグ修正はユーザーストーリーの一つとして扱っても問題ないと考えています。
  - ユーザーストーリーと同じ尺度で優先度を決めます。
- Epic
  - 複数のバックログアイテムをまとめたもの。内容が明快になっておらず、適切なサイズに分解できないものや、依存関係のあるバックログアイテムをまとめて置くための分類。
  - バックログ上で優先度が低い間はEpicとしてまとめておいて問題ないが、優先度が高くスプリントバックログに移動する前には個別のバックログアイテムに分割します。

## バックログアイテムの優先順位は誰が決めるか


バックログアイテムの優先順位は

