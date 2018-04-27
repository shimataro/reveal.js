## BigInt あれこれ

##### [Node学園 30時限目](https://nodejs.connpass.com/event/83639/) (2018/04/27)

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
* 前回も参加しました: [dynamic import あれこれ](https://shimataro.github.io/slides/20180222-nodeschool-29.html)

------

### この発表について

* 対象: Number型じゃ物足りない人
* 技術レベル: 初級〜中級

↓スライドはこちら↓

* https://speakerdeck.com/shimataro
* https://shimataro.github.io/slides/

------

### 目次

* 背景
* さっそく使ってみる
* 速度検証
    * Numberとの速度比較
* 本番環境で使うには？
* まとめ

------

## 始める前に

---

### 【慶】Node学園 30回【祝】

今後も盛り上げていきましょう！

![東京Node学園](assets/20180427-tng-30/tng30.png)

---

### 【慶】4/25 Node.js v10リリース【賀】

![Node.js v10](assets/20180427-tng-30/node-v10.png)

---

### 【拝】4/20 関西Node学園 開校【賀】

お互い盛り上げていきましょう！

![関西Node学園](assets/20180427-tng-30/kng1.png)

------

## それでは始めます

------

## 背景

---

### 背景

Number型

* 64bit（ただし **整数部は53bitまで** ）
* `9007199254740991` （約9千兆）が限界

もっと大きな整数を扱いたい！

* 全世界の負債総額は **2京円以上** だってさ
* TwitterのIDも53bitでは表現しきれない

時代は64bit

* DBにも64bit整数とかあるよね

---
<!-- .element: data-background="assets/20180427-tng-30/bg-tc39-bigint.png" data-background-position="top center" -->

### 背景

BigInt 登場！

https://tc39.github.io/proposal-bigint/

* 現在はStage3
* Node v10 でサポート
    * ただし `--harmony-bigint` が必要
* 64bitではなく **任意精度**

------

## さっそく使ってみる

---

### さっそく使ってみる

~~まずはビルド~~

```bash
$ git clone --depth 1 https://github.com/nodejs/node.git
$ cd node
$ ./configure
$ make -j4      # 並列ジョブ数はCPUのコア数に合わせる
$ ./node -v     # バージョン確認！
v10.0.0-pre
```

正式リリースされたから公式ページからインストール！

https://nodejs.org/

---

### さっそく使ってみる

* BigIntの生成
    * `100n`, `0x100n`
    * `BigInt(100)`, `BigInt("100")`, `BigInt("0x100")`
* 型変換: `Number(100n)`, `100n.toString()`
* 小数点以下は切り捨てられる: `(1n / 2n) === 0n`
* **Number型との共演はNG**
    * ビットシフトすら `1n << 10n` と書く
    * 文字列結合は `"a" + 1n` でOK

------

## 速度検証

---

### 速度検証

現在最大の[メルセンヌ素数](https://ja.wikipedia.org/wiki/%E3%83%A1%E3%83%AB%E3%82%BB%E3%83%B3%E3%83%8C%E6%95%B0#%E3%83%A1%E3%83%AB%E3%82%BB%E3%83%B3%E3%83%8C%E7%B4%A0%E6%95%B0%E3%81%AE%E4%B8%80%E8%A6%A7)（2^77232917 - 1)

```bash
$ time node --harmony-bigint -e "2n ** 77232917n - 1n"

real    0m0.110s      ←0.11秒！！
user    0m0.094s
sys     0m0.016s
```

**超速い！**

---

### 速度検証

底を変えてみる

```bash
$ time node --harmony-bigint -e "3n ** 77232917n - 1n"

real    124m47.252s   ←2時間！？
user    124m43.277s
sys     0m0.260s
```

```bash
$ time node --harmony-bigint -e "4n ** 77232917n - 1n"

real    0m0.161s      ←0.16秒！
user    0m0.132s
sys     0m0.029s
```

---

### 速度検証

10進文字列に変換してみる

```bash
$ time node --harmony-bigint -e "(2n ** 77232917n - 1n).toString(10)"

real    473m33.712s   ←8時間！！
user    473m32.767s
sys     0m0.208s
```

16進文字列に変換してみる

```bash
$ time node --harmony-bigint -e "(2n ** 77232917n - 1n).toString(16)"

real    0m0.165s      ←0.16秒！
user    0m0.125s
sys     0m0.040s
```

---

### 速度検証

結果

* 2進数で簡単に計算できるものは速い
    * 2の累乗、4の累乗
    * 16進文字列への変換
* 逆に、2進数で時間がかかるものは遅い
    * 3の累乗
    * 10進文字列への変換

考察: **内部的には2進数ベースで計算している？**

~~（考察じゃなくてソース読めよ…）~~

------

## Number型との速度比較

---

### Number型との速度比較

雑なベンチマーク

```javascript
let i, j, k, val;
for(i = 1; i <= 100; i++) {
    for(j = 1; j <= 100; j++) {
        for(k = 1; k <= 100; k++) {
            for(l = 1; l <= 100; l++) {
                val = i;
                val += j;
                val *= k;
                val %= l;
            }
        }
    }
}
```

---

### Number型との速度比較

Number

```bash
real    0m0.381s      ←0.38秒
user    0m0.352s
sys     0m0.029s
```

BigInt（数値に `n` を追加）

```bash
real    0m22.087s     ←22秒！
user    0m24.116s
sys     0m0.355s
```

Number型で収まる範囲の演算でも、BigIntだと遅い

⇒Numberを使えるなら使ったほうがいい

------

## 本番環境で使うには？

---

### 本番環境で使うには？

* 本番環境ではLTSを使いたい
    * 10月まで待つ？
* 本番環境は `--harmony` とかあまり使いたくない
    * v12まで待つ？
    * v12のLTSリリースは来年の10月？

**待ってられん！**

---

### 本番環境で使うには？

Babelはどうよ

![babel-bigint](assets/20180427-tng-30/babel-bigint.png)

https://babeljs.io/blog/2017/09/12/planning-for-7.0#stage-3-bigint-new-unfinished

---

### 本番環境で使うには？

多分こういうこと

```javascript
a += b; // a, bがBigInt型かどうかわからない
```

↓バベるとこうなる（多分）↓

```javascript
class BigInt { ... } // ラップ用のクラス

if (typeof a === "BigInt") {
    a = a.plus(b);
} else {
    a += b;
}
```

全部の演算子を置き換えなきゃアカン

---
<!-- .element: data-background="assets/20180427-tng-30/bg-big-integer.png" data-background-position="top center" -->

### 本番環境で使うには？

今はまだ `big-integer` パッケージを使おう

```bash
$ npm i -S big-integer
```

```javascript
const BigInteger = require("big-integer");

// "150"
const strValue = BigInteger(100)
    .plus(50)
    .toString();
```

* 10進数ベースの演算
    * `10000000` 単位でパックしている
* 意外と速い

------

### まとめ

* Node v10で `BigInt` に対応したよ
    * ただし `--harmony-bigint` が必要だよ
* 2進表現で高速に計算できるものは速いよ
* Numberよりはかなり遅いよ
* 現時点では `big-integer` を使おう
* 関西Node学園もよろしく！
    * 登壇者大募集！
* 手品に興味があったら声をかけてね！

------

## ありがとうございました

------

## おまけ

ここまで見てくれてありがとう( \*´艸｀)

------

## BigInt vs big-integer 2番勝負

------

### Round 1: 雑ベンチ再び

big-integer版

```javascript
const BigInteger = require("big-integer");

let i, j, k, val;
for(i = 1; i <= 100; i++) {
    for(j = 1; j <= 100; j++) {
        for(k = 1; k <= 100; k++) {
            for(l = 1; l <= 100; l++) {
                val = BigInteger(i);
                val = val.plus(j);
                val = val.multiply(k);
                val = val.mod(l);
            }
        }
    }
}
```

---

### Round 1: 雑ベンチ再び

計測結果

```bash
real    0m7.587s
user    0m8.001s
sys     0m0.362s
```

BigIntの結果（再掲）

```bash
real    0m22.087s
user    0m24.116s
sys     0m0.355s
```

**BigIntより速くね？**

実は、big-integerはNumberに収まる範囲の演算はそのまま計算するので速い

------

### Round 2: 場外乱闘

…というわけで、Number型の範囲外でもう一度挑戦してみます。

---

### Round 2: 場外乱闘

big-integer版

```javascript
const BigInteger = require("big-integer");

let i, j, k, val;
for(i = 1; i <= 100; i++) {
    for(j = 1; j <= 100; j++) {
        for(k = 1; k <= 100; k++) {
            for(l = 1; l <= 100; l++) {
                val = BigInteger(Number.MAX_SAFE_INTEGER).plus(i);
                val = val.plus(j);
                val = val.multiply(k);
                val = val.mod(l);
            }
        }
    }
}
```

```bash
real    0m41.659s
user    0m42.663s
sys     0m0.932s
```

---

### Round 2: 場外乱闘

BigInt版

```javascript
let i, j, k, val;
for(i = 1n; i <= 100n; i++) {
    for(j = 1n; j <= 100n; j++) {
        for(k = 1n; k <= 100n; k++) {
            for(l = 1n; l <= 100n; l++) {
                val = BigInt(Number.MAX_SAFE_INTEGER) + i;
                val += j;
                val *= k;
                val %= l;
            }
        }
    }
}
```

```bash
real    0m30.764s
user    0m33.004s
sys     0m0.343s
```

---

### Round 2: 場外乱闘

* BigIntのほうが1.3倍速かった
* 変換と加算が増えてBigIntさんも遅くなったけどね

------

### 結論

* Numberに収まるときもある場合はbig-integerも検討の価値あり
* 収まらないことが明確な場合はBigIntのほうがいい

------
<!-- .element: data-transition="zoom" data-background="assets/common/hiring2.svg" data-background-position="middle center" -->
