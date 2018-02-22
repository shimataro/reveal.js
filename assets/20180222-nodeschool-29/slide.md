## dynamic import あれこれ

##### [Node学園 29時限目](https://nodejs.connpass.com/event/78902/) (2018/02/22)

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

* 対象: ES2017のdynamic importに興味のある人
* 技術レベル: 中級
* Webサービス開発者には特に見てもらいたい

↓スライドはこちら↓

* https://speakerdeck.com/shimataro
* https://shimataro.github.io/slides/

------

### 目次

* 従来のimport
* 動的importの需要
* dynamic import 爆誕
* 使用上の注意点
    * 解決方法
* まとめ

------

## それでは始めます

------

## 従来のimport

---

### 従来のimport

* ES2015で標準化
* 複数の `import` 方法が定義されている

```javascript
import defaultMember from "module-name";
import * as name from "module-name";
import { member } from "module-name";
import { member as alias } from "module-name";
import { member1 , member2 } from "module-name";
import { member1 , member2 as alias2 , [...] } from "module-name";
import defaultMember, { member [ , [...] ] } from "module-name";
import defaultMember, * as name from "module-name";
import "module-name";
```

詳細は[MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import)を参照

---

### 従来のimport

使えない例

```javascript
function bar() {
    import "foo";
}

if (condition) {
    import "bar";
}

import moduleName;
```

* **グローバルスコープ** で使用
* **実行開始時点で** 必要なモジュールを確定

------

## 動的importの需要

---

### 動的importの需要

例1: セッションデータの保存先をconfigで指定したい

```yaml
# config.yaml
session:
  store: "connect-memcached"
  options:
    hosts:
      - "localhost"
    secret: "a secret key"
```
```javascript
import session from "express-session";
import Store from config.session.store;

app.use(session({
    ...,
    store: new Store(session)(config.session.options),
}));
```

---

### 動的importの需要

例2: 開発時のみ、開発支援モジュールを使いたい

```yaml
# config.yaml
develop: true
```
```javascript
if (config.develop) {
    // プロファイラやらメモリリーク検出やら
    import "some-development-support-modules";
}
```

------

## dynamic import 爆誕

---

### dynamic import 爆誕

* ES2017の新機能
* V8 ver6.3 / Chrome ver63 で実装
* Node.js ver10でサポート？

---

### dynamic import 爆誕

* 関数呼び出しスタイル
* Promiseを返す

```javascript
import("foo")
    .then((foo) => {
        // ロード成功
        ...
    })
    .catch((err) => {
        // ロード失敗
        ...
    });
```

* **named importのみサポート**

---

### dynamic import 爆誕

* 関数呼び出しスタイル
* Promiseを返す

```javascript
// async/awaitもOK
try {
    const foo = await import("foo");
    ...
}
catch(err) {
    ...
}
```

* **named importのみサポート**

---

### dynamic import 爆誕

セッションデータ保存先の指定例

```javascript
import session from "express-session";
const StorePromise = import(config.session.store);

app.use((req, res, next) => {
    StorePromise
        .then((Store) => {
            const store = new Store.default(session);

            session({
                ...,
                store: store(config.session.options),
            })(req, res, next);
        })
        .catch(next);
});
```

`import()` がPromiseを返すので、少し複雑になる

------

## 使用上の注意点

---

### 使用上の注意点

…とまあ便利なdynamic importですが、

Webサービスで使う場合には注意が必要です。

※ここから本題です

---

### 使用上の注意点

こんな状況を仮定

* http://example.com/a へのアクセスで、 `moduleA` が動的にロードされる

```javascript
app.get("/a", (req, res, next) => {
    import("moduleA")...;
});
```

* このページはまだアクセスされていない
    * つまり `moduleA` もまだロードされていない

---

### 使用上の注意点

こんな状況を仮定 その2

* リファクタリングや仕様変更などで、 `moduleA` がなくなった（リネーム・移動を含む）

```javascript
app.get("/a", (req, res, next) => {
    // モジュール名を変更
    import("moduleA2")...;
});
```

---

### 使用上の注意点

リリース手順

1. 変更後のソースをサーバに転送
    * `moduleA` が削除される
2. Node.jsを再起動

***

1と2の間で http://example.com/a にアクセス

⇒ `import("moduleA")` でエラー！

---

### 使用上の注意点

時系列で説明しよう

![エラーが起きるとき](./assets/20180222-nodeschool-29/time-series.svg)

------

## 解決方法

---

### 解決方法（？）

その1: **何もしない**

* アクセス頻度の低いページでしか起きない
* 再起動はせいぜい数秒
* そこに当たった人は運がなかったってことで。

⇒あまり重要でないページならまあアリかも？

***

* 決済のようなクリティカルなページで起きたら問題
* graceful restartは再起動までに時間がかかる

---

### 解決方法（？）

その2: **転送時、不要になったモジュールも残す**

* rsyncなら `--delete` オプションをつけずに実行

⇒Not Foundエラーは起きない

***

* モジュールの仕様変更に対応できない
* Not Foundエラーより深刻になる可能性が…

---

### 解決方法（？）

その3: **オフラインでデプロイ**

* 前段にバランサ・後段にNodeサーバ（複数）
* 1台ずつバランサから外してデプロイ

⇒中規模以上ならこんなサーバ構成になっているはず

***

* 自動化が難しい
* **クライアントサイドでの発生は防げない**

---

### 解決方法（？）

クライアントサイドでの発生例

```javascript
$("#someButton").on("click", () => {
    // ページロード後、イベント発生前にモジュールが消えたらエラー！
    import("module")...;
});
```

* 起点が **Node.js起動後** ではなく **ページロード後**
    * ユーザ数が多いほど発生率が上がる
* SPAでは発生しやすい
    * ページ滞在時間が長い（リロードしない）
    * イベントが多い
* CDN上の古いモジュールをロードする可能性

---

### 解決方法（！）

その4: **dynamic importを使わない**

* ぶっちゃけ一番簡単で確実
* 全てグローバルスコープで `import`
    * Node.js起動時に全モジュールをメモリにロード
    * その後でモジュールが削除されてもOK
* サーバサイドなら最初に全部読んだ方が効率的

---

### 解決方法（！）

dynamic importの存在意義って…

* ◎ Node.js起動時に動的ロード
    * セッションデータ保存先の指定とか
* ○ バッチ処理
    * 実行タイミングがある程度決まっている
* △ あまりクリティカルでないページ
    * エラーが出ても「ごめんね」で済むところ
* ？ Electron…で使える？
    * アップデート時に同じ問題が起きそう

------

## まとめ

* ES2017にdynamic importが導入されたよ
* 特にWebサービスで使うときは注意してね
    * ていうか使わないほうがいいかもよ
* 手品に興味があったら声をかけてね！

------

### ご清聴ありがとうございました

------

## おまけ

ここまで見てくれてありがとう( \*´艸｀)

------

### 参考資料1

[Dynamic importをウェブサービスで使うときの注意点](http://blog.zelkova.cc/2018/01/nodejs-dynamic-import.html)
* 私の記事です
* 今回の話はこれが元です

---

### 参考資料2

[V8 6.3で追加されたECMAScriptの機能](https://qiita.com/shisama/items/f7029822a5848592cbc2)
* 同僚の記事です
* v6.3で追加された3つの機能にをサンプルコードつきでわかりやすく説明しています

---

### 参考資料3

[V8 JavaScript Engine: V8 Release 6.3](https://v8project.blogspot.jp/2017/10/v8-release-63.html)
* V8チームのブログです
* ver6.3での変更点・改善点の詳説があります

------
<!-- .element: data-transition="zoom" data-background="assets/common/hiring2.svg" data-background-position="middle center" -->
