---
title: "ここ半年で読んだ技術書レビュー 2021/02"
date: 2021-02-21T14:30:00+09:00
draft: false
categories: [ "tech" ]
tags: [ "DDD", "オブジェクト指向", "アジャイル開発", "React.js" ]
---

社内LT会のスライドより抜粋。DDD,オブジェクト指向,アジャイル関連が多め。  
フロントエンドまわり触れる予定ができたのでReact関連も読んだ。  
主観多めに含んだレビューとなってます。

## [現場で役立つシステム設計の原則](https://gihyo.jp/book/2017/978-4-7741-9087-7)
- オブジェクト指向の考え方を用いて、変化に強い設計をするための実践紹介。リーダブルコード的な内容や、ドメインモデルの見つけ方、その実装方法など
- DDDと謳ってないが、やってることはDDD
- **DDD特有のわかりにくい用語が全然出てこず読みやすい**
- DDDを薦めるうえで、間違いなく一番最初に読んでほしい一冊

## [オブジェクト指向でなぜつくるのか 第2版](https://shop.nikkeibp.co.jp/front/commodity/0000/P84650/)
- 「Animal,Dog,Catを使った継承の説明」「カプセル化はgetter/setter」といったふーんで終わる説明**ではなく**、現場で使えるオブジェクト指向の考え方を解説
- 現実にあるものをオブジェクトとして捉えるのは間違い。あくまで「実現化したいことをモデル化したもの」
- アジャイル、TDDなどはオブジェクト指向から生まれた
- 大学時代まずこの本に出会えると嬉しかった…

## [ThoghtWorksアンソロジー](https://www.oreilly.co.jp/books/9784873113890/)
- タイトル通り、ThoughtWorks社のエンジニアによる短編集。紙版は絶版で入手困難、電子版はOreilly公式より購入可能
- 5章「オブジェクト指向エクササイズ」では、オブジェクト指向で書くための厳しめのルールについて解説(現場で役立つ～でも紹介あり)
- 他にもアジャイル、DDDの文脈の話がいろいろ
- (古い内容もあってきちんと読めてない…)

## [見てわかるテスト駆動開発(YouTube)](https://www.youtube.com/watch?v=Q-FJ3XmFlT8)
- (これだけ技術書ではなく動画)
- 「テスト駆動開発」の前半のコーディング手順について、書籍だと頭に入りにくい。それをライブコーディングにしたことでめちゃくちゃ理解しやすくなった
- 最後のドキュメント化されたテストコードはおおーってなった
- 「テストの構造化とリファクタリング」が大事ということが最近なってわかってきたとのこと。読みにくいテストは負債となる
 
## [レガシーコードからの脱却](https://www.oreilly.co.jp/books/9784873118864/)
- 前半では「なぜレガシーコードが生まれるのか」「技術的負債によりどれだけ多くの経済的損失が出ているか」「アジャイル的な開発には技術プラクティスも必要不可欠」など語られている
- 後半は具体的な9つのプラクティス。「なぜやるのか？」に重きを置く。『設計は最後に行う』ことの大切さなど
- 最終章の訴えがめちゃ痺れる
- エンジニアに関わらず、ソフトウェア開発に関わるすべての人に読んでほしい一冊

## [アジャイルサムライ](https://shop.ohmsha.co.jp/shopdetail/000000001901/)
- アジャイルの12の原則にのっとって、その考え方に沿ったチームのあり方や、具体的なプラクティスを紹介
- 挿絵や文体がフランクでおもしろい
- 「インセプションデッキ」「トレードオフスライダー」「バーンダウンチャート」などの考え方は有用そうで使っていきたい

## [カイゼン・ジャーニー](https://kaizenjourney.jp/)
- アジャイルをチームに浸透させていく物語ベースで、様々なプラクティスを解説
- 他の多数の書籍などから引用して有効なものを取り上げてる感じ
- いかに人を巻き込めるか、の大切さがよくわかる
- リモート環境だと難しいなーってプラクティスも多い

## [オブジェクト指向UIデザイン](https://gihyo.jp/book/2020/978-4-297-11351-3)
- 「タスクベースでUIをつくるな、オブジェクトを中心にしてそれにアクションが促せるようにしろ」
- 言いたいことは↑に尽きる、それを行うためにどういう手順を踏めばよいか？具体的な例は？などなど
- 最後の章「オブジェクト指向UIのフィロソフィー」では、なぜオブジェクト指向が生まれたか、GUIが生まれたかってところを哲学的なことも交えて語られている
- (銀の弾丸ではない。本文中でも例外でてくるし…)

## [りあクト！ 第3.1版](https://oukayuka.booth.pm/items/2368045)
- Reactについて、歴史的経緯を省かず徹底的に説明したうえで、最新の書き方などを解説
- 同人誌なこともあってかほんとに最新情報が盛り込まれてる(第3.1版は2020/12刊行)
- 1冊目はReactに入る前に、JavaScript/TypeScriptの解説だけで200p
- 3冊目は「今後はこうなっていくかも」という未来展望的な内容も面白い
- 「Reactしか勝たん」の思想がかなり強め

全体的な感想。  
オブジェクト指向をきちんと理解することで得られる保守性、拡張性についてよく学べた。またアジャイル的な観点でチームを動かすための多くのプラクティスもたくさん知れた。  
フロント・React周りについてはとにかく変化が激しく、追っていくの大変そうだという感想。

あと、ブログ記事で本の表紙画像出しにくいのどうにかしたい。。