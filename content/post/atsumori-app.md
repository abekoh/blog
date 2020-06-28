---
title: "あつ森のアイテムカタログサービスつくった、けれど公開はやめた話"
date: 2020-06-28T15:08:00+09:00
draft: false
categories: [ "tech" ]
tags: [ "Firebase", "Nuxt.js", "Vue.js", "Javascript", "個人開発" ]
---

3月ごろからあつ森にハマり、それから自粛期間に入りまして。
お家時間増えたし何か個人開発したい！と思い、思いついたのが「あつ森のアイテムをWeb上でらくらくに検索したりできるサービス」でした。

1ヶ月くらいかけてコツコツと作りあげたものの、公開しよう！ってタイミングで結局諦めることにしました。
実際どんなもの作ったか、公開やめた理由、どんな学びがあったのか、成仏のためにも記事にしようと思います。

## どんなサービス？

あつ森に出てくる家具、服などのアイテムを簡単に検索でき、そのバリエーションや入手方法などをチェックすることができます。

デモはこちら。
![atsumori-app-pc](/images/atsumori-app-pc.gif)
左側のメニューから、キーワード検索でアイテムを探すことができます。しぼりこみ検索では「ジョニーからもらえる家具」といった条件で検索ができます。
またマイリスト機能もつけていまして、ログインしていたら欲しい物を管理、公開することが可能になってます。

レスポンシブなので、スマホにも対応しています。
![atsumori-app-sm](/images/atsumori-app-sm.gif)

## なぜ公開をやめたか

やっぱり著作権的にアウトだと判断し、お蔵入りにすることにしました。

多くのあつ森攻略サイトで家具一覧ページが存在していたり、某ポケモン攻略サイトが公式からOKもらっていたりと前例があるので、
1ファンサイトとして公開してもいいかなと思ってましたが…

- 公式の画像バリバリ使ってしまってる点は、二次利用として「常識の範囲」を超えている点
- 「ゲーム買わなくてもこれ見たら満足してしまう」という不利益を与えてしまう可能性がある点

など、不安が残る点がクリアにできず。
個人的には任天堂の大ファンなので、変に迷惑かけるのも嫌でして。あくまで個人範囲で楽しむものまでに留めることにしました。

## 学びなど

諦めたものの、Webアプリをしっかり個人開発したのが初だったので多くの学びがありました。
その学びを備忘録的にメモしておこうと思います。

今回つかった技術まわりは次の通りです。

- フロント
  - Nuxt.js
    - SPAモード
    - not TypeScript
  - Vuetify
- データまわり
  - Python (アイテムデータの収集、整形、DB登録など)
- インフラ
  - Firebase
    - Authentication
    - Cloud Firestore
    - Hosting
    - (Functions)
  - Algolia (全文検索エンジン)

以下、感想など

### Vuetifyがつよい

[Vue Material Design Component Framework — Vuetify.js](https://vuetifyjs.com/)

デザインフレームワークとして今回Vuetifyを選んだのですが、強力すぎてビビりました。
CSSを全く書くことなく、HTMLテンプレ書くだけでいい感じの見た目になります。

Googleの提唱するマテリアルデザインに沿ったものなので、見た目は最近のGoogleサービスっぽくなります。
実際[レイアウトサンプル](https://vuetifyjs.com/ja/getting-started/pre-made-layouts/)もGoogleライクなのが挙げられてますね。

加えて、多く意識せずともレスポンシブになります。
当初はPCブラウザサイズでしか動作確認してませんでしたが、ふとDeveloper Toolsでスマホサイズにしてみてもほとんど崩れませんでした。  
(pxで大きめなwidth指定していても、勝手にmax-width設定されたりしてるっぽい)

きちんとレスポンシブを意識するにも、「スマホならこれくらい、タブレットならこれくらい」とサクッと設定可能です。
例えば[Grids](https://vuetifyjs.com/ja/components/grids/)を採用するとき、画面を水平に12ブロックに分けたものと考えるので

「スマホ以下だと画面いっぱい、それ以上は半分くらい」にレイアウトしたいときは
```html
<v-row>
  <v-col cols="12" sm="6">
    <v-card />
  </v-col>
</v-row>
```

と、デフォルトでは画面いっぱい`cols="12"`、スマホサイズより大きければ`sm="6"`という設定が反映されます。
これだけでいい感じにスマホ対応できるのでとにかく楽しいですね。

### 考えなしにSSRやるとハマる

当初はよく考えず「NuxtだしSSR流行ってるしそれでいくかー」という気分でSSRで作ってましたが、ハマりました。

今回Firebase Authenticationの匿名認証を使ってたんですが、それをNuxtのmiddlewareで行うと、
「サーバサイド側で認可されているもののクライアントサイドで認可されておらず、ページ遷移なしで要認証なCloud Firestoreにアクセスできない」といった事態に。

結局SSRモードでやりたいことの実現が難しそうで、SPAモードに切り替えることにしました。
行ってる処理について「サーバサイドやる」「クライアントでやる」きちんと意識して開発するのが大切ですね。

### データ周りはFirebaseのCloud Firestore

「サーバサイドエンジニアいらなくなる」という謳い文句？でよく耳にしてたFirebaseを初めて触ってみました。

今回サーバサイド側に求めるものはCRUD操作程度だったので、DBはCloud Firestoreで事足りました。
データ構造は、雑に言うとJSONをうまく拡張したものという感じ。
[公式ドキュメント](https://firebase.google.com/docs/firestore?hl=ja)のみでほぼ理解できました。

かなり限定的な用途かも？ですが、こう「かゆいところに手が届かない」って点も。

**Cloud Firestoreの複合インデックス設定で、Arrayフィールドは1つしか設定できない**

あつ森家具に紐づくラベルで、"入手方法"・"セット"は複数紐づくことがあるので、Array形式としていました。しかしインデックスは一方しかつけられないので、「"入手方法"では検索可能だが"セット"では検索できない」と機能落ちさせるしかありませんでした。

[公式ドキュメント](https://firebase.google.com/docs/firestore/query-data/index-overview?hl=ja#composite_indexes)にも注意書きされてますね。

**全文検索は他サービスに頼るしかない**

現状全文検索機能はFirebase内で完結できません。
[公式ドキュメントでも案内されている通り](https://firebase.google.com/docs/firestore/solutions/search?hl=ja)、Algolia使うのが一つの選択肢だそうです。

[Algolia](https://www.algolia.com/)

実際使ってみましたが、クライアントも多言語対応で超簡単でした。
(アイテムデータのみ検索対応だったので、Pythonで登録、JSで検索のみで使いました。)

ひらがな・カタカナ両対応させるべく、こんな風にそれぞれでインデックス登録しました。
![Algolia管理画面](/images/atsumori-app-algolia.png)

### ホスティングもFirebaseで

公開はSSRの場合はHosting + Functionsで。次の記事が参考になりました。

[SSRモードのNuxt を Firebaseにホストするまでの手順 - Qiita](https://qiita.com/sterashima78/items/394161661c634f9eda2b)

ただ前述の通り途中でSPAに切り替え、Hostingだけにしました。

### 認証認可もFirebase

Firebase Authentication、めっちゃ簡単ですね。特にGoogleログインに関しては設定一瞬です。
またセキュリティまわりを整えるため、匿名認証も簡単に設定できます。

今回は認可を受けたユーザーのみ、Cloud Firestoreのアイテムデータにアクセスできる、という設定にしたかったので
Nuxt.jsのmiddlewareにて、次のように「認可されていなければ必ず匿名認証させる」という風にしました。

```javascript
import { auth } from '~/plugins/firebase'

export default async function({ store }) {
  // storeから自分のユーザー情報取得
  const user = store.getters['users/getOwnUser']
  // 保存されていなければ匿名認証、その結果をstoreに保存
  if (!user) {
    const result = await auth.signInAnonymously()
    await store.dispatch('users/login', result.user)
  }
}
```

### Nuxtの追加モジュールが素晴らしい

ちょっとこういう機能つけたいっていうときのモジュールが豊富に用意されています。

例えば、「PWA対応させてスマホアプリっぽくしたいな」ってときはこちら入れて少し設定するだけ

[⚡ Nuxt PWA](https://pwa.nuxtjs.org/)

「サイトマップ勝手に生成してほしい」ってときはこちら

[nuxt-community/sitemap-module: Sitemap Module for Nuxt.js](https://github.com/nuxt-community/sitemap-module)

BASIC認証設定しておきたいってときはこちら

[potato4d/nuxt-basic-auth-module: Provide basic auth your Nuxt.js application](https://github.com/potato4d/nuxt-basic-auth-module)

いろいろサボれるので圧倒的感謝です。

## TypeScript入れとけばよかった

完全一人で開発してたとはいうものの、途中から脳内で「ここはこんなObject入ってくるから〜」と思考巡らせて書いてました。
多分放置して半年後はいじるのに苦労すると思います。

そもそも入れなかった理由は、以前JSもろくに触らないままTypeScript書こうとして、どこまでがJSの機能か見分けがつかなくなり苦戦したので、もうちょいJSに慣れようという思いがあったからです。
最近はようやくJS, ES6, Node.jsの記法の分別がついてきたので、TypeScript再チャレンジしたいと思います。

### 利用規約、プライバシーポリシー

これらも用意まではしました。こちらサイトを参考に、必要に応じて追加・削除してみました。

[Webサイトの利用規約（無料テンプレート・商用利用可）](https://kiyaku.jp/)

書いていくともうすぐ公開だ！とワクワクする反面、個人情報管理について緊張感が出てきます。
Firebaseで簡単になんでもできてしまう分、ほんとに大丈夫？ってなりますね。Cloud Firestoreのrules設定を何度も見直しました。

## まとめ

次はきちんと公開できるものを、よりクリーンなコードで作れたらと思いました。
そのためにもまずは、健全な(?    )アイデアがほしいですね。
引き続きフロントまわりは勉強続けていきたいと思います。