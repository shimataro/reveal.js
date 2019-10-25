## Node.js v12のES Modules

##### [関西Node学園 8時限目](https://nodejs.connpass.com/event/147459/) (2019/10/25)

<div style="text-align: right;">
  <small>小田島 太郎 / @shimataro</small>
</div>

------

### 自己紹介

![QR Code](assets/common/myqr.svg)<!-- .element: width="180" style="float:right;" -->

* 小田島 太郎 <https://shimataro.me>
* Webエンジニア（最近はインフラ寄り）
* 趣味は**手品**
  * 手品業界→Web業界
* 実は関西Node学園の立ち上げにも関わってます

------

### この発表について(1)

対象者

* ES Modulesって何？
* Node.js v12で何か変わったの？

**非**対象者

* Node.jsを触ったことないよ
* v12の変更点もちゃんとフォローしてるよ

---

### この発表について(2)

ゴール

* ES Modulesについて理解し、使えるようになる
* Node.js v12で有効なコードを書けるようになる

↓スライドはこちら↓

* <https://speakerdeck.com/shimataro>
* <https://shimataro.github.io/slides/>

------

### 目次

* きっかけ
* CommonJS
* ES Modules
* 相互運用
* v12での変更点
* 対応方法
* まとめ

------

## それでは始めます

------

## きっかけ

* **自作のnpmパッケージ**を公開中
  * CommonJS / ES Modules どちらでも使える
* CIを導入
* **Node.js v12のES Modules版**でコケていた
  * v12以外では問題なし
  * CommonJS版では問題なし

どういうこと？🤔

------

## CommonJS

---

### CommonJSおさらい

> CommonJSとは、サーバーサイドなどのウェブブラウザ環境外におけるJavaScriptの各種仕様を定めることを目標としたプロジェクトである。
(Wikipediaより)

**ECMA標準ではない**（後に一部取り込まれた）

* **モジュール管理** ←今回話すやつ
* 非同期処理
* ファイル入出力 etc

---

### Node.jsとCommonJS

Node.jsではCommonJS形式のモジュールをサポート

```javascript
// import
const fs = require("fs");

// export
module.exports = () => {
  console.log("hell,word"); // 地獄の言葉
};
```

最近は使うことは少なくなった（※個人の感想です）

------

## ES Modules

---

## ES Modulesおさらい

* **ECMAScript6(ES2015)で策定された**モジュール仕様
  * 関数や変数ではなく**構文**として導入
* BabelやTypeScriptを使うとCommonJS形式に変換してくれる
  * 今はこの方法が主流？（※個人の感想です）

```javascript
import fs from "fs";

export default () => {
  console.log("hell,word"); // 地獄の言葉
};
```

---

### Node.jsとES Modules

* **v8.5.0**から実験的サポート
  * Windowsでは**絶対パス**でimportするとエラーになるので、**v8.6.0以降**を使ってください
* 拡張子は`.mjs`
* `--experimental-modules`フラグが必要

```javascript
// v8.5.x on Windows

// "C"というURIスキームが見つからない
import foo from "C:/path/to/foo";
```

------

## 相互運用

* `module.exports`したものを`import`する場合
* `export`したものを`require()`する場合

---

### 相互運用

できんことないけどやめたほうがいい

![Issue](./assets/20191025-kng-8/cjs-esm.svg)

<small>[Native ES Modules - something almost, but not quite entirely unlike CommonJS by Gil Tayor](https://docs.google.com/presentation/d/1JKnj-HvAwkba9GMEuEqZjMPzAaZTL7j2f1n4JjOWXhs/mobilepresent#slide=id.p)</small>

------

## ここから本題

ここの記事の内容です。

Announcing a new --experimental-modules

[https://medium.com/@nodejs/announcing-a-new-experimental-modules-1be8d2d6c2ff](https://medium.com/@nodejs/announcing-a-new-experimental-modules-1be8d2d6c2ff)

------

## v12での変更点

破壊的変更がいろいろ。

---

### Node.js開発チーム内での議論

1. **ブラウザと挙動あわせよう**ず
  * ブラウザ（scriptタグ）では拡張子必須
  * Node.jsでは拡張子省略可
    * さらに`foo/index.js`は`foo`だけでimport可
1. いつまでも`.mjs`はイヤだ
  * 今後ES Modules形式が一般的になっていくはず
  * 10年後も`.js`=CommonJS形式でいいのか？
1. **現状との互換性**も確保したい

---

### ブラウザと挙動あわせようず

> By default in the new --experimental-modules, **file extensions are mandatory in import statements**: import ‘./file.js’, not import ‘./file’.

* **import文で拡張子を必須**にした
* CommonJS形式には適用されない（拡張子省略可）

```javascript
// ./path/to/foo.mjs というファイルをimportしたい
import foo from "./path/to/foo"; // NG
import foo from "./path/to/foo.mjs"; // OK
```

---

### いつまでも`.mjs`はイヤだ

> The **.cjs** extension provides a way to save CommonJS files in a project where both **.mjs and .js** files are treated as ES modules.

* `.mjs`と`.js`をES Modulesとして扱う
* CommonJSでは、新たな拡張子`.cjs`を使う

---

### 現状との互換性も確保したい(1)

いきなり`.js`をES Modulesとして扱われると困る

> Add **“type”: “module”** to the **package.json** for your project, and Node.js will treat all .js files in your project as ES modules.

package.jsonの設定で挙動を変える

* `module`: ES Modules形式とみなす
* `commonjs`: CommonJS形式とみなす （従来挙動＆デフォルト）

---

### 現状との互換性も確保したい(2)

いきなり拡張子を必須にされると困る

> However, the CommonJS-style automatic extension resolution behavior (‘./file’) can be enabled via a new flag, **--es-module-specifier-resolution=node**.

コマンドラインオプションで挙動を変える

* `node`: 拡張子や`index.js`は省略可（従来挙動）
* `explicit`: 拡張子が必須（デフォルト）

※全ファイルに適用される（パッケージ単位の指定は不可）

------

## 対応方法

Node.js v12に対応したコードの書き方

---

### 対応方法: 基本

* import対象のファイルに拡張子をつける
* `package.json`に以下を追加

```json
{
  ...
  "type": "module",
  ...
}
```

---

### 対応方法: Babel (1)

* [babel-plugin-extension-resolver](https://www.npmjs.com/package/babel-plugin-extension-resolver)を使うとよさげ
  * importの拡張子を自動付与してくれるプラグイン
  * 相対パス（`"."`で始まるパス）のみ対応

うまくいかない場合もある

例）@babel/preset-env でPolyfillが埋め込まれる場合

```javascript
// パッケージ内のファイルを直接参照するコードが生成される
// "."で始まらないので拡張子をつけてくれない
import "core-js/modules/es.array.iterator";
```

---

### 対応方法: Babel (2)

Babelに投げた [Issue #10548](https://github.com/babel/babel/issues/10548), [PR #10549](https://github.com/babel/babel/pull/10549)

![Issue](./assets/20191025-kng-8/babel-issue.png)
![Pull Request](./assets/20191025-kng-8/babel-pr.png)

取り込まれるまで`--es-module-specifier-resolution=node`でやり過ごしましょう

---

### 対応方法: TypeScript

**TypeScriptでは対応不可？** [Issue #33588](https://github.com/microsoft/TypeScript/issues/33588)

![Issue](./assets/20191025-kng-8/typescript-issue.png)

> TypeScript always **emits JavaScript code as written**, and import statements are JavaScript code so aren't changed on emit.

「拡張子つけてよ」→「TSはJSのコード部分は変更しないから無理だよ」

解決方法を知っている人がいたら教えてください。

---

### 対応方法: パッケージ作成

新しい仕様で**CommonJSとES Modulesの両方に対応したパッケージ**はどうやって作るの？

---

### 対応方法: パッケージ作成

**無理**

> Currently, **it is not possible** to create a package that can be used via both require(‘pkg’) and import ‘pkg’.

パッケージの汎用性にこだわりたい場合は、従来仕様の `"type": "commonjs"` を設定しましょう

------

## まとめ(1)

* ES Modulesの挙動は**v12から変わった**よ
  * `.js`は**ES Modules形式**だよ
  * import時に**拡張子必須**だよ
* CommonJSとES Modulesの**両方に対応したパッケージは作れない**よ
* **手品が好きな人**はお話しましょう

---

## まとめ(2)

ゴール（再掲）

* ES Modulesについて理解し、使えるようになる
* Node.js v12で有効なコードを書けるようになる
* 手品に興味を持つ

------

## ありがとうございました
