---
title: "MLサービスのServingって大変だよねという話"
emoji: "😒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ML"]
published: false
---

先日のカジュアル面談で盛り上がった話

## MLサービスのServingって大変だよね！！！
大変…なんですよ！！！
ML系のサービスはいざ本番環境で提供しようとすると

- 推論
  - リクエスト？めっちゃくるよ？👍
  - レスポンス？はやくちょーだい！！！😊
  - リソース？それなりにつかうよ～！！！👌
- 学習
  - リクエストは、推論に比べたら少ないかな。でも学習しないと精度落ちるよ？🤷‍♀️
  - レスポンスは、なるはやがいいかな～😉
  - リソース？めっっっっっっっっっっっっっっっちゃつかうよ😊😊😊
  - くそでか学習データをどうにかして持ってこないといけないんだよね🤷‍♀️
  - あ、あと…モデル管理もちゃんとしないとね？😘

という、めちゃくちゃめんどくさ激ヤバ案件なんですよ
（当然組み込まれるシステム側の要件によりますが）

当然、並列化やUXでイイ感じに待ち時間を短縮させたりすることもできますが、そういった工夫が必要になるよねってことで
普通にアプリケーションエンジニアリング的に高度な要求をさらっとしてくるんですね。やつらは。

ということでちゃんとServingさせるためにアーキテクチャからいろいろと考えないといけないことが多くて大変です。。。

## MLサービスの組み込み開発って大変だよね！！
巷ではやれつよつよMLエンジニアが最新の論文がどうだとか、なんとかGrandMasterになったとかどうだとか言ってますけどね
組み込み開発も！！大変なんですよ！！！

- ちゃんとしたロジック組む必要がある
  - 研究開発的にはこれでいけた！！！ってなっても、それをそのまま製品ソースに使えるかって言うと…ね？
- リソース効率大事
  - 無駄にリソース使ってる処理とかあるとすぐ死にます。お金との戦い。
  - 全データ一気に処理できると思った？
  - カーネルにアプリのプロセス殺されたことある？
- サービスは長く生きるので保守性だいじ
  - ちゃんとしたソース書きましょう
- ☆TEST☆
  - 書こう
- さらに…呼び出されるサービス側の仕様変更にも合わせていく！！！

ということで！！！大変なんですよ！！！

## MLサービスの研究だけやってればいいと思った？
そういう会社さんもありますけど！！！そんな人と余裕はない会社もたくさんあります！！！！

モデル改善もアプリ開発も両方やるのも…す、スキルが上がっていいと思います！
でも、どっちもやるのは…大変なんですよ！！

## でもなんか嫌いになれないんだよね
だってMLやってるって言うとなんかカッコイイじゃん？
というテンションでやってたら、なんだかんだ長いこと付き合ってました

ほぼ勢いとノリで書きました、お粗末様でした
