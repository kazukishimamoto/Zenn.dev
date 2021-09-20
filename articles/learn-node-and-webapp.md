---
title: "Node.js/Webアプリケーションに関する基礎知識メモ"
emoji: "📗"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Web", "Nodejs"]
published: true
---

## Node でよく使われる環境変数と利用方法

| 変数名     | 用途                                                                   |
| ---------- | ---------------------------------------------------------------------- |
| NODE_PATH  | モジュール検索するパスを";"区切りで指定する。                          |
| NODE_ENV   | 実行環境を指定する。                                                   |
| NODE_DEVUG | デバッグ出力したいモジュール名を指定して、デバッグ出力を有効化させる。 |

process.env から呼び出す。

```javascript
console.log(process.env.NODE_ENV);
```

## デバックで使われる用語

| 用語             | 説明                                   |
| ---------------- | -------------------------------------- |
| ステップオーバー | 関数の中へ入らずに順番に実行していく   |
| ステップイン     | 関数の中へ入って実行していく           |
| ステップアウト   | 現在の関数を呼び出し元の次の処理へ行く |

## 静的解析ツールについて

静的解析とは、コードを解析して特定のコーディングルールに違反している箇所を指摘、もしくは修正してくれるツールのこと。JavaScript では、代表的な静的解析ツールとして ESLint が挙げられる。

### ESLint の導入

以下の手順で導入する。

1. VSCode に ESLint プラグインを追加。
   VSCode 上で eslint プラグインを検索してインストールする。
2. プロジェクトに ESLint を追加する。
   `yarn add eslit@[versions] --dev` を実行してプロジェクトに追加。
   `npx eslint --init` で設定ファイルを追加。

.eslintrc.js

```javascript
module.exports = {
  env: {
    browser: true,
    jquery: true,
    commonjs: true,
    es2021: true,
    node: true,
  },
  extends: "eslint:recommended", // ESLintのおすすめルールを追加
  parserOptions: {
    ecmaVersion: 12,
  },
  rules: {},
};
```

### 静的解析のルールを設定する

[ESLint](https://eslint.org/docs/rules/indent)のドキュメントの Rules を確認し、対象の項目の設定について確認する

設定例

```javascript
module.exports = {
  // (省略)
  rules: {
    indent: ["error", 2, { SwitchCase: 1 }],
    quotes: ["error", "double"],
    semi: ["error", "always"],
    "no-unused-vars": [
      "error",
      {
        vars: "all",
        args: "node",
      },
    ],
    "no-console": ["off"],
  },
};
```

## ミドルウェアについて

リクエストレスポンスに対して任意の追加処理を行う関数。
例）リクエスト・レスポンスの変更など

:::message alert
ミドルウェアは次のミドルウェアを読んであげるために`next`が必須！
:::

```javascript
function (req, res, next) {
  // 何らかの処理
  next()
}

// エラー処理の場合
function (err, req, res, next) {
  // 何らかの処理
  next()
}

// 使うとき
app.use((req, res, next) => {
  // アプリケーションのミドルウェア
})
```

## テンプレートエンジンについて

HTML のひな形に対して動的な値や HTML タグを埋め込める仕組み。

今回はわかりやすい Effective JavaScript template(ejs)を利用する。

### ejs の基本構文

| 種類                   | サンプル | コメント                       |
| ---------------------- | -------- | ------------------------------ |
| 値出力（エスケープ）   | <%= %>   | 基本的にこれを使う（XSS 対策） |
| 値出力（案エスケープ） | <%- %>   | インクルードに限定して使う     |
| コード実行             | <% %>    |                                |
| コメント               | <%# %>   |                                |

### インクルード

erb の partial 的な概念。
何度も使われるコンポーネントを読み込んで使う。

```ejs
<%- include(path [, locals]) %>
```

引数
path: インクルードしたいテンプレートのファイルパス
locals: インクルードするテンプレートに引き渡すデータ

戻り地
生成された HTML

## log4js

log4js 内で出てくる主要概念は以下の 3 つ。

| 概念     | 説明                                                                                                                |
| -------- | ------------------------------------------------------------------------------------------------------------------- |
| Logger   | ログ出力を行う実態。<br>Logger インスタンスのメソッドを使ってログ出力を行う。<br>ex.) logger.error('error message') |
| Category | ログを分類するためのカテゴリ。<br>分類ごとにどこへ出力するかアペンダーを指定する。                                  |
| Appender | ログ出力方法。<br>ConsoleAppender、FileAppender など具体的な出力先定義。                                            |

### アクセスログ出力のフォーマット指定子

| 識別子                 | 出力内容                             |
| ---------------------- | ------------------------------------ |
| :data[format]          | 現在日時。clf/iso/web を指定。       |
| :http-version          | リクエスト HTTP バージョン。         |
| :method                | リクエストメソッド。                 |
| :referrer              | リクエストのリファラ。               |
| :req[header]           | 指定したリクエストヘッダー。         |
| :res[header]           | 指定したレスポンスヘッダー。         |
| :response-time[digits] | レスポンス時間。                     |
| :status                | レスポンスステータス。               |
| :url                   | リクエスト URL。                     |
| :user-agent            | リクエスト元の User-Agent ヘッダー。 |
