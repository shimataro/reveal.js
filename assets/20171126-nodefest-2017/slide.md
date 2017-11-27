### レスポンシブから逆戻り!?Webサービスのマルチデバイス対応方法

##### [東京Node学園祭2017](http://nodefest.jp/2017/) Day #2 (2017/11/26)
<div style="text-align: right;">
    <small>小田島 太郎 / @shimataro</small>
</div>

------
<!-- .element: data-background="assets/common/bg-intro-cards-and-coins.png" data-background-position="middle center" -->

## 自己紹介
* [小田島 太郎](http://blog.zelkova.cc/) / [shimataro@GitHub](https://github.com/shimataro) / [odashima.taro@Facebook](https://www.facebook.com/odashima.taro) / [shimataro999@Twitter](https://twitter.com/shimataro999)
* ウェブリオ株式会社所属（京都）
* サーバサイドエンジニア
* 趣味は手品

------

## この発表について
* 対象: マルチデバイス対応に苦労している人
* 技術レベル: 初〜中級
* 実戦向けの話です

↓スライドはこちら↓

https://speakerdeck.com/shimataro
https://shimataro.github.io/slides/

------

## それでは始めます

------

## マルチデバイス対応してますか？

------

## ３つの選択肢
1. 対応しない
2. デバイスごとに出力を変える
3. 1つの出力ですべて対応

歴史とともに説明します。

---

### 1. 対応しない
* デバイスを決め打ち
* かつてはPC向けサイトしかなかった
    * This page is written in Japanese only.
    * 踏み逃げ禁止！キリ番ゲッターは要報告！
* 今でもやりようによっては使える

---

### 2. デバイスごとに出力を変える
* UAからデバイスを判別して、異なるHTMLを出力
    * 判別方法は割愛
* ガラケーの普及に伴い主流に

---

### 3. 1つの出力ですべて対応
* レスポンシブウェブデザイン
    * Googleが推奨
    * スマホの普及によって注目
* 表示はクライアント側で調整

---

### そして今…
まだレスポンシブデザインで大丈夫？

------

## レスポンシブデザインの問題
* ガラケーに対応できない
* 出力内容は最大公約数的なものになる
* 本格的にやろうとするとメンテナンスは意外と面倒
* **Google AMP**

------

## Google AMPについて
![Google AMP](./assets/20171126-nodefest-2017/google-amp.png)<!-- .element: width="400" style="float:right;" -->
* Accelerated Mobile Pages
* HTML/CSS/JSの仕様を制限
* 一瞬で表示！

AMP専用のHTMLが必要

------

## お分かり頂けただろうか
* レスポンシブデザインは万能ではない
* モバイル端末こそ別の出力が必要なときもある
* 回帰現象が起きているとでもいうのだろうか

------

## ようやく本題
* 同じURLでもリクエストごとにHTMLを変えたい
* Node（Express）ではどうやる？

------

## ExpressでのUA別出力方法
* 標準では用意されていない
* ビューを `res.render()` 時に切り替えるのが一般的？

```
views
├index.pug
├index-smartphone.pug
└index-tablet.pug
```

* ページ数×デバイス数のビューが並ぶ

------

## やりたいこと
* デバイスごとに階層化

```
views
├default
│└index.pug  <- ビューはデバイスディレクトリ内に配置
├smartphone
│└index.pug
└tablet      <- ビューがないデバイスでは規定のビューを表示
```

* ソースコードの変更は最小限で！

------

## ミドルウェア作りました
https://github.com/shimataro/express-view-switcher
```bash
npm install -S express-view-switcher
```

---

### 使い方1: マルチデバイス対応
```javascript
import viewSwitcher from "express-view-switcher";

const app = express();
app.use(viewSwitcher((req) => {
    // "User-Agent" ヘッダを解析して検索したいディレクトリを列挙
    // ※実際の判別ロジックはREADME参照
    // https://www.npmjs.com/package/express-view-switcher

    // 最初に見つかったビューを表示
    // 1. views/smartphone
    // 2. views/default
    return [["smartphone", "default"]];
}));

// あとは普通に res.render() をコール
```

---

### 使い方2: 多言語対応
```javascript
app.use(viewSwitcher((req) => {
    // "Accept-Language" ヘッダやクエリストリングなどを解析
    // ※実際の判別ロジックは（ｒｙ
    // 1. views/en-us
    // 2. views/en
    // 3. views/ja
    return [["en-us", "en", "ja"]];
}));
```

---

### 使い方3: 多言語＋マルチデバイス
```javascript
app.use(viewSwitcher((req) => {
    // 組み合わせてもOK
    // ※実際の（ｒｙ
    // 1. views/en-us/smartphone
    // 2. views/en-us/default
    // 3. views/en/smartphone
    // 4. views/en/default
    // 5. views/ja/smartphone
    // 6. views/ja/default
    return [
        ["en-us", "en", "ja"],
        ["smartphone", "default"],
    ];
}));
```

------
<!-- .element: data-background="assets/20171126-nodefest-2017/bg-weblio-eikaiwa.png" data-background-position="top center" -->

## 導入してみた
Weblio英会話 https://eikaiwa.weblio.jp/

Perl→Node.jsに書き換え

※完了したとは言ってない

※全ページマルチデバイス対応とは言ってない

------

## まとめ
* デバイスごとに出力を変えなきゃいけない状況はまだまだあるよ
* ミドルウェア作ったよ
* 多言語対応もできるよ
* 手品に興味があったら声をかけてね！

------

### ご清聴ありがとうございました

------

## おまけ

ここまで見てくれてありがとう( \*´艸｀)

------

## Q&A

---

### Q: 対応バージョンは？
A:
* Node.js 4以上
* Express 4以上
* 動作中のNodeバージョンに合わせて最適化する素敵仕様！

---

### Q: `require` できないよ！
A:
```javascript
// NG: これだと使えない
const viewSwitcher = require("express-view-switcher");
```
```javascript
// OK: 時代はES6！babelで変換しよう
import viewSwitcher from "express-view-switcher";
```
```javascript
// OK: 宗教上の理由でrequireしか使えない方はこちら
const viewSwitcher = require("express-view-switcher").default;
```

---

### Q: ベースディレクトリを指定したい！
A:
```javascript
// 第二引数を指定すると、実際に使われたディレクトリが
// res.locals オブジェクトのkey/valueとして追加される。
// includeやextendのベースディレクトリを指定する際に使える。

// 例: res.locals.basedir = "views/smartphone"
app.use(viewSwitcher((req) => {
    // ※（ｒｙ
    return [["smartphone", "default"]];
}, "basedir"));

// ただし、 "view engine" が "pug" の場合は
// 自動的に "basedir" が設定されるので指定不要
```

---

### Q. `vary` ヘッダを指定したい！
A:
ごめんなさい。まだできてません。

------
<!-- .element: data-transition="zoom" data-background="assets/common/hiring2.svg" data-background-position="middle center" -->
