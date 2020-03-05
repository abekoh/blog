---
title: "Cloud Storage経由でCloud Buildを動かしてデプロイする"
date: 2020-03-05T23:01:03+09:00
draft: false
categories: [ "tech" ]
tags: [ "GCP", "Cloud Functions", "Cloud Build", "Cloud Storage", "Node.js" ]
---

GCPサービスの1つ、[Cloud Build](https://cloud.google.com/cloud-build)はいわゆるCI/CDツールです。
GithubなどGitレポジトリに紐づけてトリガーさせるのが普通で、UIからもそのような設定しかできません。

しかし、業務のある都合で[Cloud Storage](https://cloud.google.com/storage)経由でデプロイしたいことがありまして。
そのやり方を模索してみてうまくいったので、メモっておきます。

## 全体像
![全体像](/images/gcp-deploy-functions-from-storage.svg)

今回はhello-funcというファンクションを、tar.gzに圧縮したソースからCloud Functionsにデプロイするというシナリオとします。
デプロイ対象はApp Engineとか、別のサービスでもいけると思います。

## デプロイ対象
シンプルにHTTPリクエストおくるとHelloが返ってくるFunctionです。Node.jsで書きます。
```javascript
// index.js
exports.hello = (req, res) => {
  res.send('Hello, world!');
}
```
これと、`npm init`で生成したpackage.jsonを含んだhello-func.tar.gzを作っておきます。
```bash
tar zcvf hello-func.tar.gz index.js package.json
```

## ビルドファンクション実装
メインとなるビルド用ファンクションです。

Storageのイベントをトリガーとして発火させ、Cloud BuildのREST APIを叩いてビルド、デプロイを実行します。
