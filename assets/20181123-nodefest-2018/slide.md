## おまいらちゃんと<br/>リソース解放してますか

##### [東京Node学園祭2018 Day #1](http://nodefest.jp/2018/) (2018/11/23)

<div style="text-align: right;">
  <small>小田島 太郎 / @shimataro</small>
</div>

------
<!-- .element: data-background="assets/common/bg-ejje.png" data-background-position="top center" -->

### 自己紹介

* [小田島 太郎](http://blog.zelkova.cc/) / [shimataro@GitHub](https://github.com/shimataro) / [odashima.taro@Facebook](https://www.facebook.com/odashima.taro) / [shimataro999@Twitter](https://twitter.com/shimataro999)
* ウェブリオ株式会社所属（京都）
* サーバサイド / インフラエンジニア
* 趣味は手品
  * 手品業界→Web業界
* 最近は **手打ちうどん** にハマりかけてます

------

### この発表について

* 対象
  * Node.jsで **Webアプリ** を作っている人
  * 他の言語からの転入生
  * 特に **リクエストごとにプロセスやメモリ空間が独立している世界** からの来訪者（e.g., PHP）
* 技術レベル: 中級

↓スライドはこちら↓

* <https://speakerdeck.com/shimataro>
* <https://shimataro.github.io/slides/>

------

### 目次

* 最初に
* リソースの管理方法
  * その1
  * その2
  * その3
* どうすればいいの？
* まとめ

------

## それでは始めます

------

## 最初に

---

### リソースとは

**ここで言う** リソースとは、プログラムが確保・解放する必要がある **メモリ以外の** 資源を指す

* ファイルディスクリプタ
* コネクション

特に **(R)DBコネクション** についてのお話です

メモリはGCが回収してくれるので今回の対象外

------

## リソースの管理方法

DBコネクションをどうやって管理してますか？

---

### 1. グローバルオブジェクト

```javascript
const mysql = require("mysql");

// これを使いまわす
const connection = mysql.createConnection(...);
```

グローバルなリソースを使いまわす

---

### 1. グローバルオブジェクト

利点

* リソースは最小限
* 毎回確保する必要がないので高速

欠点

* 切断時の再接続処理が必要
  * 瞬断
  * MySQLでは8時間アイドル状態だと切断される
* トランザクションが他のリクエストを巻き込む
  * **開発時は気づきにくい**

---

### 2. 都度確保・都度解放

```javascript
const mysql = require("mysql");
const pool = mysql.createPool(...);

function someFunc() {
  try {
    const connection = pool.getConnection(); // 必要になったら確保
    ... // 何か処理
  } finally {
    connection.release(); // 使い終わったら解放
  }
}
```

必要なときに確保、不要になったら解放

---

### 2. 都度確保・都度解放

利点

* リクエスト間のトランザクションは独立する

欠点

* 1リクエスト中に **無駄に** リソースを2つ以上作成する場合あり（関数のネスト等）

---

### 3. リクエスト内で使いまわす

```javascript
const mysql = require("mysql");
const pool = mysql.createPool(...);

function getConnection(req) {
  if(req.connection === undefined) {
    // reqにリソースを関連付ける
    req.connection = pool.getConnection();
  }

  return req.connection;
}
```

リクエストオブジェクトにリソースを関連付け、このリソースを使いまわす

（もしくはContinuation Local Storageを使用）

---

### 3. リクエスト内で使いまわす

利点

* 1コネクション/リクエスト 以下

欠点

* 解放忘れに注意
  * **リクエスト処理の最後** で解放を忘れると…

---

### 3. リクエスト内で使いまわす

こうなります

![connections](./assets/20181123-nodefest-2018/connections.png)

（転入生がハマる罠）

---

### 3. リクエスト内で使いまわす

転入生のハマりどころ

* リクエストを捌き終えても **プロセスは走り続ける**
  * 強制解放されない
  * PHPとは違うのだよ
* デストラクタ/ファイナライザがない
  * GCでメモリは回収されるが、 **それ以外のリソースは回収されない**

------

## どうすればいいの？

しばらく悩みました。

* 正常終了・異常終了で **飛ぶイベントが異なる**
* F5連打されると **イベントが飛ばない** （ことがある）
* リソース取得自体が非同期だとさらにややこしい

---

### こうしとけ

```javascript
function middleware(req, res, next) { // Express.jsのミドルウェア
  req.connection = ...;

  res
    .on("finish", () => {
      req.connection.release(); // 正常完了時
    })
    .on("close", () => {
      req.connection.release(); // 異常終了時（通信切断）
    });
  if (res.socket.destroyed) {
    // すでに通信が切断されていた（イベントが発生しない）
    req.connection.release();
  }
}
```

* 3箇所で解放
* 特に最後は大事（F5連打対策）

---

### 簡単な方法

`on-finished` パッケージを使うと楽
<https://www.npmjs.com/package/on-finished>

```javascript
const onFinished = require("on-finished");

function middleware(req, res, next) { // Express.jsのミドルウェア
  req.connection = ...;

  onFinished(res, () => {
    req.connection.release(); // ここで解放するだけ！
  });
}
```

---

### ちなみに…

グローバルオブジェクトを使いまわす場合は

* 切断時にしっかり再接続して
* トランザクション時に新しくリソースを作る

うまくやってくれるORMを使おう
（e.g., Sequelize）

でも、処理完了のタイミングを知っておくのも役に立ちます

------

## まとめ

* リソース管理には注意しよう
  * 他の言語から来た人は特に注意
* 処理完了の検知は **3箇所** で！
  * **パッケージ** を使うと楽だよ
* **ORM** を使うともっと楽だよ
* **うどん** が好きな人は声をかけてね！

------

## おまけ

---
<!-- .element: data-background="assets/20181123-nodefest-2018/bg-kng.png" data-background-position="top center" -->

### 関西でもNode学園！

* [関西Node学園 梅田キャンパス 1時限目（04/20）](https://nodejs.connpass.com/event/82614/)
* [関西Node学園 梅田キャンパス 2時限目（07/05）](https://nodejs.connpass.com/event/89037/)
* [関西Node学園 3時限目（08/03）](https://nodejs.connpass.com/event/94758/)
* [関西Node学園 4時限目（11/02）](https://nodejs.connpass.com/event/102985/)

東西で盛り上げていきましょう！

------

## ありがとうございました
