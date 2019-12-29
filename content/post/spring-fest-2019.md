---
title: "Spring Fest 2019 参加レポート"
date: 2019-12-30T02:05:02+09:00
draft: false
categories: [ "tech" ]
tags: [ "Spring", "Java", "勉強会" ]
---

12/18に開催されたSpring Fest 2019に参加してきました。
その時聞いたものについてのレポートです。

## 基調講演：From Spring Boot 2.2 to Spring Boot 2.3
[スライド](https://www.slideshare.net/makingx/from-spring-boot-22-to-spring-boot-23-jsug)

Spring Boot 2.2での目玉機能、2.3で追加される予定のもの、Spring Cloudについての紹介でした。
気になったものいくつか取り上げます。

#### Health Indicator Group
ヘルスチェックのエンドポイントを複数、グループ別で設定できる機能。
```
management.endpoint.health.group.liveness.include=ping
management.endpoint.health.group.readiness.include=db,redis
```
↑の設定の場合、
`/actguator/health/liveness`と`/actuator/health/rediness`が提供される。

サンプルの設定どおり、KubernetesのLiveness/Readiness Probeと相性よさげ。

#### Immutable Configuration Properties
コンストラクタインジェクションでCongiurationPropertiesが設定可能に。
→Setter不要。

またKotlinだとdata classにも適用できるとか。

#### GraalVM
Javaだけどネイティブアプリ化させたりできるやつ。
昨年のSpring Festでは出たばっかりのころだったけど、順調にサポートが厚くなってきている感じ。

2020年に出るSpring 5.3になると、諸々設定が楽になるとのこと。

#### Cloud Native Buildpacks
ソースコードを解析してOCI標準イメージを作成するツール。

(OCI標準イメージとは、Open Container Initiativeが定めた標準仕様に沿ったコンテナイメージのこと。 https://www.publickey1.jp/blog/17/open_container_initiativeoci_v10.html)

packというCLI使って簡単に試せる。  
https://github.com/buildpacks/pack

特にKubernetes向けの場合、kpackというのも提供されている。
https://github.com/pivotal/kpack

## Spring Boot爆速開発超絶技巧
[スライド](https://speakerdeck.com/yusuke/spring-boot-and-intellij-idea-technique)

ひたすらIntelliJのショートカット・機能を紹介していただくというセッション。
結構知らないショートカットもあって為になりました。

使えそうと思ったのが、breakpointのオプション。
「この条件に一致したらbreak」とか「breakせずにprintだけ行う」とかできる、知らなかった。。

## LINE公式アカウントのチャットシステムにおけるSpringおよびWebFluxの活用事例
[スライド](https://speakerdeck.com/line_developers/examples-of-using-spring-and-webflux-in-the-chat-system-for-line-official-accounts)

Spring WebFluxを実際にプロダクトに適用した事例紹介。

WebFluxを利用することで、各リクエストをノンブロッキングで処理することで、1つのイベントループスレッドで複数リクエストを捌けるようになる。そしてサーバー数が減少につながる。

実際うまく稼働できているとのこと。このあたり理解が浅いと感じたので、実際に触ってみようと思います。

## NissanConnectの舞台裏で動くSpring Boot／Spring Cloud 〜Microserviceの実運用の事例〜
[スライド](https://www.slideshare.net/DaisukeMorishita1/spring-boot-and-spring-cloud-inside-nissanconnect-at-spring-fest-19)

日産車とスマホアプリなどとを連携するシステムの裏側紹介と、Springに関するTips紹介でした。

Tipsで引っかかりそうと思ったのが、pacheHttpClientのデフォルト接続数問題。  
デフォルトだと同一ドメインに対して同時リクエスト2つまでしかできないので、`maxConnPerRoute`を設定する必要があるとのこと。実際ぶち当たると苦戦しそうですね。。

## Spring Developer のための コンテナ入門
Google Cloudさんによる、コンテナ、Kubernetesについて入門的な内容でした。

ここで紹介されていた、Jibはかなり使えそうな印象。

https://github.com/GoogleContainerTools/jib

Javaアプリのコンテナイメージを、Dockerfileいらずで作ってくれる優れもの。
Mavenプラグインも用意されていて、mvnコマンドだけでdocker build, docker pushまでできるみたい。
CI/CDでも設定削減できたり相性よさそうな気がします。

## Quarkus による超音速な Spring アプリケーション開発
[スライド](https://www.slideshare.net/ChihiroIto1/quarkus-spring)

RedHatさんによるQuarkusの紹介でした。

Quarkusとは、Kubernetesなどのコンテナ環境に最適化されたJavaアプリを実現するフレームワークのこと。

フレームワーク起動時に、設定ファイル解析、クラスパス・クラスのスキャン、リフレクションの準備などJava実行時に毎回行う、時間かかるやつを一度だけ行うようにして、起動時間短縮・メモリ使用量削減につなげている。

また、従来のJITコンパイラ向けでも、GraalVMでネイティブイメージ向けにも両方利用できて、両方ともそれなりに恩恵を受けられるとのこと。

## Efficient Web Apps with Spring Boot 2

その場でSpring MVC -> Spring WebFluxに置き換えるライブコーディングでした。

非同期化することでUI表示が目に見えて速くなるのが面白かった。

## Spring with React for Enterprise Application
[スライド](https://speakerdeck.com/sdaigo/spring-with-react-for-enterprise-application)

業務アプリについて、変化できるUI、変化に強いシステムを作り上げる手法について、タグバンガーズさんがやっていることの紹介でした。

- イベントストーミングというモデリング手法の利用。「何が起こったか」「なぜ起こったか」といったことを付箋に書き、それらの関係性をホワイトボードに書いていって、境界づけなど行ってモデリングしていく手法。シンプルでとっつきやすそうでした。
- UIではCSS-in-JSを利用。CSSがどう継承しているかとか分からなくなる事態を防ぐ。JSにうまく組み込むことでコンポーネントごとに管理できるように。
- BEでは、イベントストーミングで分離した対象それぞれ、Springのエコシステムどれ使うか？を考える。それぞれ境界間の関係性に最適なのを選んでいく。
- テストも、付箋の「○○ならば××」というのをそのままテストにできる。
- 結合テストのモック化は、Spring Cloud Contract, Pactの組み合わせがおすすめとのこと。

実際業務アプリ開発に多く関わっているので、参考にしてみたい部分が多くありました。

## 全体通しての感想
今回よく聞いたキーワードとしては"Reactive"と"Native Image"ですかね。
どちらもCloud Nativeの流れに沿うようJava, Springを発展させようという動きなのでしょうか。

また個人的には、昨年に比べて「これ業務に活かせそう」という観点ができて楽しかったです。また来年も来よう。あと月次でやってるようなイベントにも参加してみたい所存です。
