---
title: "Hugo, Github Pages, CircleCIつかってブログ構築"
date: 2019-12-14T13:40:05+09:00
draft: false
tags: ["Hugo", "CircleCI", "Github Pages", "Google Domains"]
---

このブログの構築メモ。
やっぱりブログもGitHubで管理できたらいいなーと探したら、この組み合わせで簡単にいい感じにできそうだったのでやってみた。


## Hugoとは

https://gohugo.io/

Go製の静的サイト生成ツール。
とにかく簡単にブログがつくれる。ブログじゃなくてもポートフォリオサイトやOSSプロジェクトページなんかもいける。

Markdownでかけるのも嬉しい。非常にGitHubフレンドリーな感じ。

個人的に惜しいと思う点は、超スタンダードな感じのテーマの多くがGPLなところ。
編集中のはPrivateにする場合ここが引っかかってしまうので、なくなくそれらを弾いてテーマ選びました。

## 構成

![全体構成](/images/circleci-github-hugo.png)

Github Pagesの機能をつかって公開するんですが、Hugoのソース自体はPrivateで管理。
abekoh.github.ioにはCircleCIがmaster pushするだけ。

## CircleCI設定


## ドメイン設定
