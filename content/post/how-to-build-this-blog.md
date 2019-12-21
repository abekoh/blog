---
title: "Hugo, Github Pages, CircleCIつかってブログ構築"
date: 2019-12-14T13:40:05+09:00
draft: false
categories: [ "tech" ]
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

とりあえずこれを無編集で使ってます。

~~https\://github.com/zwbetz-gh/vanilla-bootstrap-hugo-theme~~

→ これに変更しました https://github.com/achary/engimo

## 構成

![全体構成](/images/circleci-github-hugo.png)

Github Pagesの機能をつかって公開するんですが、Hugoのソース自体はPrivateで管理。
abekoh.github.ioにはCircleCIがmaster pushするだけ。

CircleCI選んだ理由は、とりあえず有名でモダンなやつ試したかったから。
最近だとGithub Actionsがよかったかな、と後から思ったけどまぁいいか。

## CircleCI設定

hugoのビルドは、Orbがあったのでそれを使う。
Orbはビルド手順のテンプレートみたいなの。

https://circleci.com/orbs/registry/orb/circleci/hugo

その後、Github Pagesへのpushは手動で設定。
このあたり参考にさせていただきました。

- [CircleCIでgithub pagesに自動デプロイする](https://qiita.com/sterashima78/items/ddb8161eb6345d9fb15b)
- [CircleCIでHugoを実行してGitHub Pagesにデプロイ](https://t32k.me/mol/log/hugo-circleci-ghpages-2018/)

引っかかったのが、ssh鍵設定してもcloneできない問題。
[こちら](https://discuss.circleci.com/t/git-clone-fails-in-circle-2-0/15211)参考に、`StrictHostKeyChecking=no`にすれば解決しました。

最終的に.circleci/config.ymlはこんな感じ。
なれてきたらまた直していきたい。

```yaml
version: 2.1
orbs:
  hugo: circleci/hugo@0.4.1
jobs:
  deploy:
    docker:
      - image: cibuilds/base
    steps:
      - add_ssh_keys:
          # CirlceCIのSSH Permissionsに設定したSSH Keyのfingerprintを設定
          fingerprints:
            - "SO:ME:FIN:G:ER:PR:IN:T"
      # ビルドしたworkspaceをもってくる
      - attach_workspace:
          at: .
      - deploy:
          name: deploy to Github Pages
          command: |
            # ssh警告無視
            echo "HostName github.com" >> ~/.ssh/config
            echo "StrictHostKeyChecking no" >> ~/.ssh/config

            DEPLOY_DIR=deploy
            # USER_EMAILはCircleCIのEnvironment Variablesで設定
            git config --global user.email $USER_EMAIL
            git config --global user.name $CIRCLE_USERNAME
            git clone git@github.com:abekoh/abekoh.github.io.git $DEPLOY_DIR

            cd $DEPLOY_DIR
            rm -vrf ./*
            cp -v -R ../public/* ./

            # ドメイン設定
            echo "blog.abekoh.dev" > CNAME

            git add -f .
            git commit -m "Deploy #$CIRCLE_BUILD_NUM from CircleCI [ci skip]"
            git push origin master -f
workflows:
  version: 2.1
  main:
    jobs:
      - hugo/build:
          # TODO: HTMLチェックをonに
          html-proofer: false
      # masterマージ時のみデプロイ
      - deploy:
          requires:
            - hugo/build
          filters:
            branches:
              only: master
```

## ドメイン設定

ついでにドメインも取得してみたので設定。

Google Domainsで、devドメインつくりました。
年間1200円、安いですね。

![Google Domains設定](/images/google-domains-cname-config.png)
このようにCNAME設定して、

![Github Pages](/images/github-pages-domain-config.png)
abekoh/abekoh.github.ioのSettingsでドメイン設定するだけ。

なお、CNAMEファイルがpush時に毎回消えてしまうような設定になっているので、.cricleci/config.ymlのpipelineのとおりCNAMEを毎回作成するようにしています。

## 感想
やっぱりGitHub上で完結できるのよいですね。書きたいネタをIssueに登録、PR作って解決という流れで一人で運用ができて楽しい。
