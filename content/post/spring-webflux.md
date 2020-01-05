---
title: "Spring Webfluxを試す"
date: 2019-12-31T17:43:49+09:00
draft: true
---

Spring Fest 2019でよく出てきたので、試してみようってやつ。

## 公式ドキュメントみてみる
https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux

- なぜできたの？
  - 1. 少ないハード資源でスケースさせるために、少ないスレッドで動作するノンブロッキングなWebスタックの需要が高まってきたため。
  - 2. 関数型プログラミングを利用できるようにしたいため？
- MVCみたくアノテーションベースでも、ラムダベースでも可能
- 通常のJDBCはブロッキングなので使えない、R2DBCつかおう
- デフォでNettyだけど、TomCatも選べる
  - でもTomcatの使い方全然違う

## 資料
- https://speakerdeck.com/shintanimoto/introduction-to-reactive-programming-using-spring-webflux?slide=9
- https://spring.io/guides/gs/reactive-rest-service/
- https://medium.com/w-logs/jdbc-for-spring-webflux-spring-data-r2dbc-99690208cfeb … MySQLつかうサンプル
- https://blog.jetbrains.com/idea/2019/12/tutorial-reactive-spring-boot/ … jetbrainsによるチュートリアル
- https://kazuhira-r.hatenablog.com/entry/20180811/1533997040
- https://netjs.blogspot.com/2018/11/spring-web-reactive-webflux-example-functional-programming.html
- https://mike-neck.hatenadiary.com/entry/2018/03/04/002523 ifempty
- https://medium.com/@nikeshshetty/5-common-mistakes-of-webflux-novices-f8eda0cd6291 よくある間違い
- https://projectreactor.io/docs/core/release/reference/index.html#which-operator
- https://stackoverflow.com/questions/55594738/how-to-retrieve-a-generated-id-when-saving-objects-with-spring-data-r2dbc-using
test
- https://howtodoinjava.com/spring-webflux/webfluxtest-with-webtestclient/
