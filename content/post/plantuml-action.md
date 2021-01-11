---
title: "GitHub Actions + PlantUMLでドメインモデルの管理を楽にする"
date: 2021-01-11T22:30:00+09:00
draft: false
categories: [ "tech" ]
tags: [ "DDD", "GitHub Actions", "PlantUML" ]
---

ドメインモデルの図を複数人、エンジニア・ドメインエキスパート間で共有するにあたって、
やり方はいろいろ考えられますが、

- 差分がわかりやすい
- バージョン管理ができる

という点からやはりGitで管理できると嬉しいと思います。

Gitで管理しやすいフォーマットとして、DSLから図を自動生成してくれるPlantUMLが使いやすく定番です。
ドメインモデルについては下図のようにオブジェクト図で書けるとよいかと思います。

![ドメインモデル図の例](/images/domain-model.png)
(『実践ドメイン駆動設計』 P.356, 図10-7をもとに作成)


**このPlantUMLによるドメインモデルの管理をGitHubのPR上で効率行えるようにするための、Github Actionsのテンプレを開発しMarketplaceに公開しました。**

[Generate and Commit PlantUML Diagrams · Actions · GitHub Marketplace](https://github.com/marketplace/actions/generate-and-commit-plantuml-diagrams)

使い方などはREADMEを参考に。使い勝手は以下のサンプルPRを見ていただければわかりやすいと思います。

[Example by abekoh · Pull Request #33 · abekoh/commit-plantuml-action](https://github.com/abekoh/commit-plantuml-action/pull/33)

.pumlファイルに差分が生じたとき、同じところに.pngとして画像が生成され、コミットされます。

さらに`enableReviewComment`を有効にすれば、生成された図のリンクとプレビューがコメントに投稿されます。
Beforeを展開して古い図との差分も確認可能です。

![Review comment sample](/images/commit-plantuml-action.png)

毎回リンクをたどってレビューを行う必要がなく、使い勝手よく扱えると思います。


注意点としては、PlantUMLが依存しているgraphvizの影響でライセンスがGPLとなっている点です…  
[このあたり](https://plantuml.com/ja/vizjs)のPlantUMLの別実装を使ってより緩いライセンス使えないか、模索していきたいところ。