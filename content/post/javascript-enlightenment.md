---
title: "『開眼! JavaScript』読んだ"
date: 2020-02-23T18:20:11+09:00
draft: false
categories: [ "tech" ]
tags: [ "JavaScript", "読書" ]
---

Oreillyから出てる『開眼！ JavaScript ――言語仕様から学ぶJavaScriptの本質』の読書メモです。

https://www.oreilly.co.jp/books/9784873116211/

仕事でもWebフロントアプリを触ることがあるものの、既存のコードを雰囲気で改修する程度で、今ひとつJSの挙動など理解できていなかったのでその辺りが知りたく。
ちょうどこの本が「JavaScript特有の癖、落とし穴」にフォーカスを当てていて、大変参考になりました。

以下、刺さったところの個人的メモ。

### JSは基本なんでもオブジェクト、プリミティブでもオブジェクトのように扱える

number, string, boolean, null, undefinedはプリミティブ、他はすべてオブジェクト扱い。
Array、Functionも結局Objectを着色したようなイメージ。

プリミティブであっても、それに対応するラッパーオブジェクト(numberならNumber)のプロパティ、メソッドが呼び出せる。呼び出したとき、そのときだけオブジェクトを生成→破棄という流れになる。

```javascript
'hoge'.length // -> 4
```

### JSのオブジェクトはミュータブル、基本なんでも挙動変えられる

Arrayなどネイティブコンストラクタを持つオブジェクトであっても、windowなどグローバルオブジェクトであっても、そのプロパティ・メソッドは書き換え可能。
`window.alert()`でも書き換えて機能停止させることも可能。varつけず`foo = 'bar'`とやればグローバルオブジェクトのプロパティをいじったことになる。

もちろん、可能なだけで破壊的な変更は推奨されない。

### thisはそれを呼び出すタイミングで指すものが決まる

```javascript
var foo = 'foo';
var myObject = { foo: 'I am myObject.foo' };

var sayFoo = function() {
	console.log(this['foo']);
};

// myObjectのsayFooメソッドにsayFoo()関数を与える
myObject.sayFoo = sayFoo;

myObject.sayFoo(); // myObject.sayFoo()内でのthisはmyObjectなので'I am myObject.foo'を出力
sayFoo(); // sayFoo()内でのthisはグローバルオブジェクトなので'foo'を出力
```
(P.87より引用)

定義時の'foo'が常に出ると予想されるかもだが、実際は実行時のコンテキストに依存する。

### 無名関数の即時実行

```javascript
(function(){ console.log('hoge'); })()
```
functionの中身を即実行する書き方。
何度かこんなコードみたことあったのに、意味理解できていなかった。。

### 関数の巻き上げ

function後ろのほうで定義、前のほうで実行でも問題なし。
```javascript
foo(); // -> hoge
function foo() { console.log('hoge'); }
```
これを関数の巻き上げと呼ぶ。JS特有ですね。
関数内で外側の変数つかってたりすると引っかかりそう。
(参考: [やっとわかったjsの「巻き上げ」 - Qiita](https://qiita.com/39_isao/items/d9d80e98b5bd1938bc1d))

### コンストラクタのprototypeはインスタンスでは__proto__で参照できる

Chrome Developer Toolsでオブジェクトについてる`__proto__`プロパティ、正体はそれを生成したコンストラクタの`prototype`への参照でした。
要するに、`myObj.__proto__`と`myObj.constructor.prototype`は同じものを指す。

この仕様は標準ではないらしい、けれど殆どのブラウザで機能している。

### numberのラッパーオブジェクトのメソッド呼び出し

本文に直接無い内容だけど、自分理解のため。
数字をカンマ区切りにするとき、`toLocalString()`が使えるけど実際どう呼び出すんだろって色々ためすと
```javascript
10000.toLocalString() // -> エラー
'10000'.toLocalString() // -> '10000'
10000..toLocaleString() // -> '10,000'
(10000).toLocaleString() // '10,000'
Number(10000).toLocaleString() // -> '10,000'
new Number(10000).toLocaleString() // -> '10,000'
var a = 10000
a.toLocaleString() // -> '10,000'
```
という具合。`10000.toLocalString()`だと数値として評価できていない。
3行目の`..`となるのは、最初のドットは小数点として評価されるため。

### その他
- プロトタイプチェーンについて図も使ってかなりじっくり解説されてわかりやすかった。ブログにまとめるの大変なので割愛。  
  (Web記事だとこちらがわかりやすそう: [や...やっと理解できた！JavaScriptのプロトタイプチェーン](https://maeharin.hatenablog.com/entry/20130215/javascript_prototype_chain))
- 便利ライブラリとしてUnderscore.jsが紹介されているが、今だとLodashのほうが主流かな？
- `Math.PI`といった定数は変更不可。円周率3にできたりしない。

### 感想
他にも色々へぇ〜となるポイントだらけでした。業務だとTypeScriptだけど、こういう癖をうまく吸収したりしてくれて助かっている反面、やっぱりきちんと理解しておかないと詰まりそうだなーという点が多々。非常に勉強になりました。

それなりにフロントも見れるように、さらにJS勉強していきたい所存です。
