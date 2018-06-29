## CJSとESMとnpmパッケージ

##### [Node学園 31時限目](https://nodejs.connpass.com/event/90936/) (2018/06/29)

<div style="text-align: right;">
    <small>小田島 太郎 / @shimataro</small>
</div>

------
<!-- .element: data-background="assets/common/bg-intro-cards-and-coins.png" data-background-position="middle center" -->

### 自己紹介

* [小田島 太郎](http://blog.zelkova.cc/)
    * [shimataro@GitHub](https://github.com/shimataro)
    * [odashima.taro@Facebook](https://www.facebook.com/odashima.taro)
    * [shimataro999@Twitter](https://twitter.com/shimataro999)
* ウェブリオ株式会社所属（京都）
* サーバサイドエンジニア
* 趣味は手品

------

### この発表について

* 対象: **汎用** npmパッケージを開発している人
* 技術レベル: 中級
    * npmパッケージやBabelの基礎知識を前提
    * 技術的に濃い話ではありません

↓スライドはこちら↓

* https://speakerdeck.com/shimataro
* https://shimataro.github.io/slides/

------

### 目次

* 背景
* 今回のゴール
* CJSとESM
* パッケージの構成
* コード生成
* まとめ

------

## 始める前に

---

### 7/5 関西Node学園 2時限目

![関西Node学園](assets/20180629-tng-31/kng2.png)

------

## それでは始めます

------

## 背景

---

### 背景

JavaScript(ECMAScript)のモジュール
* `require()` / `module.exports` (CommonJS; CJS)
* `import` / `export` (ES Modules; ESM)

ネイティブESMはあまり使われていない
（BabelでCJSに変換するのが一般的）

でもきっとそのうち普及する

------

## 今回のゴール

---

### 今回のゴール

CJS / ESM / Babelの **どれからでも使える** パッケージモジュールを作りたい

```javascript
// パッケージ "foo"
export default foo;
export {bar};
```

↓

```javascript
// CJS
const {default: foo, bar} = require("foo");
```

```javascript
// ESM / Babel
import foo, {bar} from "foo";
```

---

### 今回のゴール

パッケージの条件
* ソースコードは単一
    * ビルド時に複数ファイルの生成はOK
* default export / named exports 両方対応
* ソースコード内に変なハックは入れない
    * **おまじない** NG
    * **古い文法** NG

作る側にも使う側にも、極力負担をかけない

------

## CJSとESM

---

### CJSとESM

拡張子 `.mjs` はESM
```javascript
export default foo;
export {bar};
```

それ以外の拡張子はCJS
```javascript
module.exports = foo;

// または
exports.default = foo;
exports.bar = bar;
```

---

### CJSとESM

相互運用について

![CJSとESMの相互運用](assets/20180629-tng-31/cjs-esm.svg)

<small>[Native ES Modules - something almost, but not quite entirely unlike CommonJS by Gil Tayor](https://docs.google.com/presentation/d/1JKnj-HvAwkba9GMEuEqZjMPzAaZTL7j2f1n4JjOWXhs/mobilepresent#slide=id.p)</small>

---

### CJSとESM

* 1つのファイルで全部対応するのはややこしい
* `.mjs` と `.js` を両方用意する方が確実
* 最初にESMで書いて、CJS形式にも変換

**Babelの出番**

------

## パッケージの構成

---

### パッケージの構成

http://yosuke-furukawa.hatenablog.com/entry/2016/05/10/111102

* `package.json` 内の `main` を **拡張子なし** で指定
```json
{
  ...
  "main": "./index", // ".js" はつけない
  ...
}
```
* 2種類のindexを用意
    * `./index.js` （CJS/Babel用）
    * `./index.mjs` （ESM用）

------

## コード生成

---

### コード生成

概要

`index.js` はこうする

```javascript
exports.default = foo;
exports.bar = bar;
```

`index.mjs` はこのままでOK

```javascript
export default foo;
export {bar};
```

---

### コード生成 - CJS

`index.js` は普通にBabelで変換すればOK

```javascript
export default foo;
export {bar};
```

↓

```javascript
exports.default = foo;
exports.bar = bar;
```

---

### コード生成 - ESM

`index.mjs` は何もしなくてOK？

* Nodeがサポートしている文法だけを使うならOK
    * デコレータとかstatic propertiesとか使えない
* **汎用** パッケージなら、ある程度古いバージョンもサポートしたい
    * せめてLTSくらいは…
    * そのために **古い文法** を使うのは嫌だ

というわけで、 **やっぱりBabelは使いたい**

---

### コード生成 - ESM

Babel使用時の注意 その1
* `babel-preset-env` のデフォルトはCJS形式
* import / export構文を変換しない場合はこうする

```json
// .babelrc
{
  "presets": [
    ["env", {"modules": false}]
  ]
}
```

---

### コード生成 - ESM

Babel使用時の注意 その2
* 拡張子が自動的に `.js` になる
    * ESMでは `.mjs` じゃないとダメ
* Babel7に `--keep-file-extension` が登場
    * 拡張子を変更しないオプション
    * 変換元の拡張子も `.mjs` にする必要あり
    * まだbeta段階

---

### コード生成（ESM）

選択肢は2つ
* Babel単体で解決
    * Babel7 ＆ `--keep-file-extension`
* 他のツールを導入
    * 変換後に拡張子をリネーム

**今回は後者（gulp）を採用**
* Babel6（安定版）でも使える
* 元の拡張子に制限なし

------

## 実際に作ってみた

---

### 実際に作ってみた

https://github.com/shimataro/hell-word

```bash
npm install @shimataro/hell-word
```

ESM(.mjs) / Babel
```javascript
import foo, {bar} from "@shimataro/hell-word";

foo(); // "hell, word"
bar(); // "地獄の言葉"
```

CJS
```javascript
const {default: foo, bar} = require("@shimataro/hell-word");

foo(); // "hell, word"
bar(); // "地獄の言葉"
```

------

### まとめ

* 単一ソースからCJS/ESM/Babel対応のパッケージを作る方法
* まずESMの `export` 構文で作る
    * CJS用は `exports.default` に変換→ `.js`
    * ESM用はそのまま→ `.mjs`
* `package.json` の `"main"` に拡張子をつけない
* サンプル作ってみたからよかったら参考にしてね
* 関西Node学園もよろしく！
* 手品に興味があったら声をかけてね！

------

## ありがとうございました
