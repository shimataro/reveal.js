## NPMパッケージをDeno対応してみた(Deno勉強会 9)

##### [Deno勉強会 9](https://scrapbox.io/deno-ja/Deno%E5%8B%89%E5%BC%B7%E4%BC%9A9)

<div style="text-align: right;">
  <small>小田島 太郎 / @shimataro</small>
</div>

------

### 自己紹介

![QR Code](assets/common/myqr.svg)<!-- .element: width="180" style="float:right;" -->

* 小田島 太郎 <https://shimataro.me>
* 関西在住
* Webエンジニア
  * フロントエンド / バックエンド / インフラ
* 趣味は**手品**
  * 手品業界→Web業界
  * 以前は**某マジシャン**のもとで仕事

------

### 最初に

Deno歴4日なので、もっと簡単に解決できる方法があるかもしれません。

何か知っていたら教えてください。

------

### 目標(1/2)

自作のnpmパッケージをDeno対応したい: <https://www.npmjs.com/package/value-schema>

* 入力が規定の書式に沿っているかを検証して、必要に応じて修正するライブラリ
	* `username`: 20文字以下の文字列
    * `limit`: 1以上100以下の整数 / 省略時は50 / 数値文字列が渡されたときは数値に変換 / 100を超えたら100にする
* もともとTypeScriptで書いている

---

### 目標(2/2)

Denoで実行するときの**import文の拡張子問題を何とかする**のが今回の目的

* `.ts`をつけたらNPM向けのパッケージをビルドする時にtscに怒られる
* つけないとDenoで使えない
* **同一ソース**でNPMとDenoの両方に対応したモジュールを作りたい

------

### アプローチ

* ソースファイルは拡張子なしで`import`（tscでビルドできるように）
* ビルドステップでDeno向けの**拡張子つきソースを別途出力**
* `import` / `export from`対応

**そのためのNPMツールを作る**

------

### 成果

<https://www.npmjs.com/package/deno-module-extension-resolver>

```bash
# インストール
npm i -g deno-module-extension-resolver
```

```bash
# 使い方
deno-module-extension-resolver SRC_DIR DST_DIR
```

* `SRC_DIR`以下にある`.ts`/`.js`ファイル内の`import`モジュール名の拡張子を解決
* ディレクトリ構造を保ったまま`DST_DIR`に出力
* npmパッケージのビルドステップでこれを走らせればよさそう

------

### 内部処理

1. AST構築
1. `import` / `export from`文からインポート対象のファイル名を取り出す
1. 既に同名のファイルがあったら何もしない
1. ファイル名に`.ts`, `.js`, `/index.ts`, `/index.js`の順でサフィックスをつけて、そのファイルが存在したらそのファイル名に置き換える
1. どれにも一致しなかったら警告を表示

---

### 例

拡張子解決されるもの

```typescript
import foo from "/absolute/path/to/module";
import bar from "./relative/path";
import "/path/to/dir"; // ディレクトリ（index.ts, index.jsがある場合）
import * as baz from "/named/export";
export * from "/export/aggregation";
export {qux, quux} from "/export/aggregation";
```

拡張子解決されないもの

```typescript
import "/already/resolved.ts";
import "/files/to/resolve/not/exists";
import "neither/absolute/nor/relative";
import "https://example.com/not/file/path";
require("/deno/does/not/support/require");
```

------

### 自作NPMパッケージに適用してみた結果

Deno公式ページのサードパーティーモジュールに登録した
<https://deno.land/x/value_schema>

```typescript
import vs from "https://deno.land/x/value_schema/mod.ts";
```

モジュール名にハイフンが含まれていると登録できないので、アンダースコアにした

------

### 結局使えそう？(1/2)

* サードパーティー製のモジュールを使わない、自己完結するパッケージならこれでOK
    * 今回のパッケージは自己完結している
* サードパーティー製のパッケージまで効力は及ばない
    * 大改造になりそうだし`node_modules`に依存しそう

**限られた範囲でしか使えない**

---

### 結局使えそう？(2/2)

サードパーティー製パッケージを使う場合

`import`文で問題があるなら、`import`文をなくせばいい（逆転の発想？）

**Webpackで1ファイルにバンドルしたら解決した**

* fsとかのNode標準モジュールは使えません
* JSで出力されるから**型チェックができない**
* Webpackを使う場合は依存パッケージに**GPL**が含まれていたらGPLにしなきゃいけない？

------

### まとめ

* 拡張子解決するモジュールを作ったよ
    * <https://www.npmjs.com/package/deno-module-extension-resolver>
    * 自己完結するモジュールしか対応できないよ
* Deno対応したモジュールを登録してみたよ
    * <https://deno.land/x/value_schema>
* サードパーティー製ライブラリに依存する場合はWebpackでバンドルするといいよ
    * **型チェックできない**かもよ
    * **ライセンスまわり**は注意が必要？

---

### ありがとうございました

![QR Code](assets/common/myqr.svg)<!-- .element: width="360" -->

Follow me!
