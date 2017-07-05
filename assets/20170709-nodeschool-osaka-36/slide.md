# ES6で始めるNode.js
### 〜注目をあびる優れた開発手法〜

##### [NodeSchool Osaka #36](https://nodejs.connpass.com/event/60303/) (2017/07/09)
<div style="text-align: right;">
    <small>小田島 太郎 / @shimataro</small>
</div>

------
<!-- .element: data-background="assets/20170709-nodeschool-osaka-36/bg-intro.png" data-background-position="middle center" -->

## 自己紹介
* [小田島 太郎](http://blog.zelkova.cc/) / [shimataro@GitHub](https://github.com/shimataro) / [odashima.taro@Facebook](https://www.facebook.com/odashima.taro) / [shimataro999@Twitter](https://twitter.com/shimataro999)
* ウェブリオ株式会社所属（京都）
* 趣味は手品
* 昨日はBBQでリア充ごっこしてきました

------

## この発表について
レベル
* 初級〜中級

キーワード
* ECMAScript6 (ES6)
* Babel
* babel-preset-env

------

## それでは始めます

------

## JavaScriptのつらいところ
* functionだらけ
    * クラスもfunction
    * コールバックのたびにfunction
* 非同期処理
    * 流れを追いづらい
    * コールバックのネスト地獄

もっと使いやすくならんかな…

もうJavaScriptなんて進化しないよな…

------
<!-- .element: data-transition="zoom" -->

## してた。
ECMAScript
* JavaScriptの規格
* 現行の「いわゆるJavaScript」はECMAScript5(ES5)
* ECMAScript6(ES6)ではもっと便利に！
* 他のAltJS(TypeScript, CoffeeScript, ...)と一線を画す
    * JavaScriptある限りECMAScriptは不滅
    * ES6の構文を一部取り入れているブラウザもある

------

## ES6の機能
* `import` / `export`（モジュール）
* `class`（クラス定義）
* `let` / `const`（変数・定数）
* `for of` 構文（配列のループ）
* `async` / `await`（非同期処理 ※ES7）
* テンプレート文字列
* アロー演算子
* デフォルト引数…などなど。

---

## ES5 vs ES6（モジュール）
```javascript
// ES5
// ただの関数呼び出し？
var foo = require("foo");

// ただの代入？
module.exports = bar;
```
```javascript
// ES6
// インポートだ！
import foo from "foo";

// エクスポートだ！
export default bar;
```

---

## ES5 vs ES6（クラス定義）
```javascript
// ES5
// クラスに見えない
function aClass() {
    // コンストラクタ
}
aClass.prototype.aMethod = function() {
    // aClassのメソッド
}
```
```javascript
// ES6
// クラスだ！
class aClass {
    constructor() {
        // コンストラクタ
    }
    aMethod() {
        // aClassのメソッド
    }
}
```

---

## ES5 vs ES6（変数・定数）
```javascript
// ES5
var variant = 0;
function foo() {
    console.log(variant); // undefined（変数巻き上げ）
    {
        var CONSTANT = 1; // 変えるなよ！絶対変えるなよ！
        var variant = 1;
    }
    console.log(CONSTANT); // ここでもアクセスできる
}
```
```javascript
// ES6
let variant = 0;
function foo() {
    console.log(variant); // 0
    {
        const CONSTANT = 1; // 変えるとエラー！
        let variant = 1;
    }
    console.log(CONSTANT); // エラー！
}
```

---

## ES5 vs ES6（配列のループ）
```javascript
// ES5
var data = [1, 2, 3];
for (var i = 0; i < data.length; i++) {
    var datum = data[i];
    console.log(datum * datum);
}
```
```javascript
// ES6
const data = [1, 2, 3];
for (const datum of data) {
    console.log(datum * datum);
}
```

------

## ES6対応状況（Node.js）
* 最新LTS(バージョン6)ではあらかた対応
* バージョンによっては、 `"use strict"` を指定しないと使えない機能がある
* サーバサイドでは古いバージョンを使わざるを得ない場合がある
    * Ubuntu 16.04ではバージョン4をサポート

------

## ES6対応状況（ブラウザ）
* モダンブラウザではある程度対応
* ただしブラウザによって差が激しい
* 誰がアクセスするかわからないウェブサービスで使うのは勇気がいる

------

## 開発時に大混乱
対応状況がバラバラなので…
* 「この文法ってNode4で対応してたっけ？」
* 「この文法ってIE9で対応してたっけ？」
* 「この文法って…」

⇒開発に直接使うのは現実的ではない

------

## やりたいこと
* 開発時はES6を使う
* ES5に変換してブラウザやNode.jsで実行

そんな都合のいいツールなんてあるわけないよな…

------
<!-- .element: data-transition="zoom" data-background="assets/20170709-nodeschool-osaka-36/bg-babel.png" data-background-position="top center" -->

## あった。
Babel - https://babeljs.io/
* ES6をES5に変換するツール（トランスパイラ）
* 由来は旧約聖書に出てくる「バベルの塔」
* プラグイン機能
    * 変換する文法を指定できる
    * 変換が必要な文法のみ柔軟に対応できる！

⇒対応状況を気にせずES6で書ける！

------

## preset
プラグインを1つずつ指定するのは面倒
* *preset* ＝ いくつかのプラグインをまとめたもの
* babel-preset-es2015: `class`、`for of`等
* babel-preset-es2017: `async`/`await`等
* とりあえず全部のpresetを入れれば動く

でもネイティブ対応している文法はそのまま使いたい
（特にNode.jsでは！）

------

## やりたいこと
ターゲットが対応していない文法だけ変換してほしい
* 例1: Node.js 4
* 例2: IE9以上、Chrome/Firefox最新版
* 例3: 現在実行中のNode.js

そんな都合のいいpresetなんてあるわけないよな…

------
<!-- .element: data-transition="zoom" data-background="assets/20170709-nodeschool-osaka-36/bg-babel-preset-env.png" data-background-position="top center" -->

## あった。
[babel-preset-env](https://babeljs.io/docs/plugins/preset-env/)
* [ES6対応一覧表](https://kangax.github.io/compat-table/es6/)を参照して、非対応文法のみ変換
* バージョン指定方法が神
    * 現在実行中のNode.jsのバージョン
    * Chromeの最新から2バージョン前
    * IEのシェア1%以上のバージョン
* presetはこれさえ覚えておけばOK！
    * 実験的な文法は別途プラグインが必要な場合あり

------
<!-- .element: data-background="assets/20170709-nodeschool-osaka-36/bg-babel-preset-env-sample.png" data-background-position="top center" -->

## 使い方
サンプルコードを用意しました
* https://github.com/shimataro/babel-preset-env-sample
* 詳しくは [gulpfile.babel.js](https://github.com/shimataro/babel-preset-env-sample/blob/master/gulpfile.babel.js) を参照

------

## 注意
* 新しい文法はそのままでは使えない場合あり
    * 特定プラグインの有効化・無効化が必要
    * `async` / `await`, `static` プロパティ等
* 新しいクラスやメソッドはpolyfillが必要な場合あり
* ランタイムライブラリが必要な場合あり
    * ブラウザ向けには[webpack](https://webpack.github.io/)等で1ファイルにまとめるのが一般的

------

## ES6の拡張子は？
* `.js` - 個人的には使いたくない
    * ブラウザに食わせていいものだけ `.js` にしたい
* `.es` - ECMAScriptの正式な拡張子
    * あまり浸透していない
* `.es6` - 正式ではないけど割と浸透している
    * バージョン6限定っぽく見える

結論: 好きなの使え

------

## まとめ
* ES6は便利だよ
* 対応状況マチマチだから[Babel](http://babeljs.io/)で変換するといいよ
* [babel-preset-env](https://babeljs.io/docs/plugins/preset-env/)が便利だよ
* ただしいくつか注意することがあるよ
* 手品に興味があったら声をかけてね！

------

### ご清聴ありがとうございました
