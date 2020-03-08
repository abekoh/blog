---
title: "Cloud Storage経由でCloud Buildを動かしてデプロイする"
date: 2020-03-08T23:30:00+09:00
draft: false
categories: [ "tech" ]
tags: [ "GCP", "Cloud Functions", "Cloud Build", "Cloud Storage", "Node.js" ]
---

GCPサービスの1つ、[Cloud Build](https://cloud.google.com/cloud-build)はいわゆるCI/CDツールです。
GithubなどGitレポジトリに紐づけてトリガーさせて使うのが普通だと思います。トリガーとしても現状、リポジトリ経由しか選択できません。

しかし、業務のある都合で[Cloud Storage](https://cloud.google.com/storage)経由でデプロイしたいことがありまして。
やり方を模索してみてうまくいったので、メモっておきます。

## ソース
今回使ったものすべてこちらに置いてます。

[abekoh/gcp-deploy-from-storage](https://github.com/abekoh/gcp-deploy-from-storage)

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
Storageのイベントをトリガーとして発火させ、Cloud BuildのAPIを叩いてビルド、デプロイを実行します。

Node.js用のGoogle APIクライアントとして、普通はこちらを使いますが

[googlespis/google-api-nodejs-client](https://github.com/googleapis/google-api-nodejs-client)

[READMEにも書かれている通り](https://github.com/googleapis/google-api-nodejs-client#working-with-google-cloud-platform-apis)、GCP上で利用する場合は下記のライブラリのほうが使い勝手いいみたいです。

[googleapis/google-cloud-node](https://github.com/googleapis/google-cloud-node)

Cloud Buildの依存を追加します。

```bash
npm init
npm i -S @google-cloud/cloudbuild
```

ビルド用Functionsの実装はこちら。
```javascript
// index.js
'use strict';
const {CloudBuildClient} = require('@google-cloud/cloudbuild');

exports.build = async file => {
    // file.metageneration: メタ情報が更新されるとインクリメントされる値
    // ファイル自体が更新されるときは'1'となる
    if (file.metageneration !== '1') {
        return;
    }
    // Promiseを返却することで、解決させてfunctionsが終了する
    return new CloudBuildClient().createBuild({
        projectId: process.env.GCP_PROJECT_ID,
        build: {
            source: {
                storageSource: {
                    // バケット名
                    bucket: file.bucket,
                    // ファイル名
                    object: file.name
                }
            },
            steps: [
                {
                    "name": "gcr.io/cloud-builders/gcloud",
                    "args": [
                        "functions",
                        "deploy",
                        "hello-func",
                        "--entry-point=hello",
                        "--runtime=nodejs10",
                        "--memory=128MB",
                        "--region=us-central1",
                        "--trigger-http"
                    ]
                }
            ]
        }
    });
};
```

[createBuild](https://googleapis.dev/nodejs/cloudbuild/latest/v1.CloudBuildClient.html#createBuild)というメソッドで、任意のオプションからCloud Buildを起動させます。
このリクエスト内ではリポジトリだけでなくCloud Storageを指定することでき、今回やりたかったことが達成できます。

また、リクエストにプロジェクトIDを含める必要があります。
Node.js 8系までは `GCP_PROJECT` という環境変数で取得できたようですが、Node.js 10系では取得できなくなったようです。

[環境変数の使用 | Google Cloud Functions に関するドキュメント](https://cloud.google.com/functions/docs/env-var#environment_variables_set_automatically)

仕方ないので、デプロイ時のコマンドラインで指定することにしました。
以下のコマンドでbuild-funcをデプロイします。

```bash
gcloud functions deploy build-func \
  --entry-point=build \
  --runtime=nodejs10 \
  --memory=256MB \
  --region=us-central1 \
  --trigger-bucket=src-func \
  --set-env-vars GCP_PRJECT_ID={GCPプロジェクト名}
```

`--trigger-bucket`には、デプロイしたいファイルを置くバケットを指定します。

## Cloud Buildの権限設定
Cloud Buildを使って対象を初めてデプロイする場合、権限設定が必要です。
今回はCloud Functionsをデプロイするのでその開発者と、サービスアカウントユーザーが必要なので付与します。

![Cloud Buildの権限設定](/images/cloud-build-settings.png)

## 動作確認
準備が整ったところで、Storageの対象バケットにアップロードしてみます。

```bash
gsutil cp hello-func.tar.gz gs://src-func/hello-func.tar.gz
```

このように、正常hello-funcがデプロイされました。

![Functionsの結果](/images/result-hello-func.png)

## まとめ
ちょっと面倒ではありますが、ちょっと工夫するだけで実現できたので良かったです。
こんな感じで、トリガーに設定できなくてもAPIが存在すればFunctionsでなんとかなるってことは他にもあるかもですね。
