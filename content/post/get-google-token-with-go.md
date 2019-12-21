---
title: "Google API の OAuth 2.0トークン取得するツールつくった"
date: 2019-12-21T13:52:48+09:00
draft: false
categories: [ "tech" ]
tags: [ "Go", "Google Photos" ]
---

Google PhotosのAPIつかって自動アップロードするバッチつくってみたくて調べたところ、
GCPサービスみたくサービスアカウント使う、ってことはできないみたいだった

https://developers.google.com/photos/library/guides/authentication-authorization#service-accounts

OAuth 2.0の認証フローをたどる必要があるみたい。

とはいえバッチつくるとき、初回のみ認証→次からはrefresh tokenで再利用という流れになるんだろうけど
初回のみ認証の実装をケチりたかったので、refresh token取得まで簡単に取得できるツールみたいなの書いてみた。

[abekoh/get-google-tokens](https://github.com/abekoh/get-google-tokens)

1. GCPサイドバー->APIとサービス->認証情報 より、認証情報を作成->OAuthクライアントID を選択
2. 用途に応じた種類を選ぶ。任意の名前をつける。JS生成元・リダイレクトURIは http://localhost:8080 を指定
![Google API OAuth 2.0設定](/images/setup-google-oauth.png)
3. 認証情報一覧から、ダウンロードボタンを押す。client_secret_XXX.apps.googleusercontent.com..jsonのようなjsonファイルが手に入る。
4. token取得ツールインストール
```bash
go get github.com/abekoh/get-google-tokens
```
5. 次を実行。例では、Google Photos APIにアップロードする権限のみ。
```
get-google-tokens -json client_secret_XXX.apps.googleusercontent.com.json -scope https://www.googleapis.com/auth/photoslibrary.appendonly
```
photoのスコープはここ参照
https://developers.google.com/photos/library/guides/authentication-authorization

6. 実行するとURLが表示されるので、アクセス。そしてスコープを許可。「このアプリは確認されていません」と表示されても進める。
7. リダイレクトされてlocalhostに移った後、ターミナルのほうを確認。Access TokenとRefresh Tokenが表示されている。

仕組みとしては、リダイレクト時にURLパラメータに`code=`と認証コードが入るので、
それをWebサーバで受け取って、チャンネル送信して、POSTリクエストでトークン取得するという流れ。

Go久しぶり書きましたが、こんな感じでWebサーバー立ち上げ簡単にできる点、CLI化も楽な点がよいですね。

## 参考
- [Google API OAuth2.0のアクセストークン&リフレッシュトークン取得手順 2017年2月版 - Qiita](https://qiita.com/iwaseasahi/items/2363dc1d246bc06baeae)
