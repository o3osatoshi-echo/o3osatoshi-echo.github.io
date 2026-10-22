---
layout:        post
title:         "モノレポ設定 tsconfig/tsup/biome/eslint"
date:          2025-09-22 00:00:00 +0900
modified_date: 2025-09-22 00:00:00 +0900
categories:    [tech, conf]
tags:          [coding]
---

![thumbnail](/assets/2025-09-22-monorepo-config/thumbnail.png)

{% include coding-notice.md %}

## config

モノレポ運用で必要な設定をパッケージで管理している。

- [npm](https://www.npmjs.com/package/@o3osatoshi/config)
- [GitHub](https://github.com/o3osatoshi/portfolio/tree/main/packages/config)

### コンセプト

- 共通設定を `@o3osatoshi/config` に集約し、モノレポ全体で型・ビルド・リントを一貫管理する
- 各アプリはベース設定を継承し、必要な差分だけを上書きすることでメンテナンスコストを抑える

### 役割

- tsconfig ... 型チェック
- tsup ... ビルド＆バンドル
- biome ... フォーマット＆リント
- eslint ... ソート


## tsconfig

型チェックのみに利用する。
共通の設定を `base.json` にまとめ、各パッケージ・アプリケーションで継承する。

### base.json

#### 厳格性に関する設定

厳格さをできる限り高める。

| 設定                                   | 解説                                                                   |
|:-------------------------------------|:---------------------------------------------------------------------|
| `noUncheckedIndexedAccess`           | 添字アクセスの未定義性を明示<br>`arr[i]` や `obj[key]` の結果型に `undefined` が付き取りこぼし検出 |
| `exactOptionalPropertyTypes`         | オプショナルと `undefined` を厳密に区別<br>「存在しない」と「存在して `undefined`」の混同を防止       |
| `noPropertyAccessFromIndexSignature` | インデックスシグネチャ経由の安易なアクセスを禁止<br>マップ/辞書型での誤アクセスを型で検出                      |
| `noImplicitThis`                     | `this` の暗黙 `any` を禁止<br>コールバック等での文脈ミスを抑止                             |
| `noImplicitReturns`                  | すべての分岐で `return` を要求<br>値の返し忘れを検出                                    |
| `noFallthroughCasesInSwitch`         | `switch` の意図しないフォールスルーを禁止<br>`break` 書き忘れによるバグを防止                    |
| `noImplicitOverride`                 | 派生クラスでの上書きに `override` を必須化<br>親の変更で別メソッド扱いになる事故を防ぐ                  |
| `noUnusedLocals`                     | 未使用ローカル変数をエラーに<br>デッドコードを早期発見。意図的未使用は `_` 始まりに                       |
| `noUnusedParameters`                 | 未使用の関数引数をエラーに<br>API 設計を明確化。意図的未使用は `_` 始まりに                         |
| `useUnknownInCatchVariables`         | `catch (e)` を `unknown` とする<br>例外を型安全に絞り込みできる（型ガード必須）                |
| `forceConsistentCasingInFileNames`   | ファイル名の大文字小文字ゆれをエラー化<br>OS 差によるビルド/実行不一致を防止                           |
| `resolveJsonModule`                  | `*.json` を型付きで import 可能に<br>JSON 構造の型検査・補完が有効に                      |
| `esModuleInterop`                    | CJS/ESM の相互運用を改善<br>`default` 周りの非互換を緩和しつつ型整合                        |
| `isolatedModules`                    | 各ファイル単体で安全にトランスパイル<br>`const enum` 等を禁止しバンドラ互換性を確保                   |
| `verbatimModuleSyntax`               | import/export の書き換え最小化<br>ツリーシェイク精度向上・不要コード混入防止                      |
| `skipLibCheck`                       | 外部型定義の再検査をスキップ<br>ビルド高速化。ただし外部 `.d.ts` の不整合は見逃す                      |

#### 互換性に関する設定

ESMをベースとしつつ、CJSやバンドラ挙動とも整合を取る。

| 設定                                 | 解説                                                                                                 |
|:-----------------------------------|:---------------------------------------------------------------------------------------------------|
| `esModuleInterop`                  | CJS と ESM の相互運用を改善（`default` 周りの扱いを円滑化）<br> 既存の CJS ライブラリを `import foo from "lib"` で扱いやすく、互換性の罠を回避 |
| `isolatedModules`                  | 各ファイル単位で安全に変換できる制約を導入（バンドラ互換）<br> esbuild/tsup/Babel 等との相性が安定し、部分ビルドや並列ビルドで破綻しにくい                  |
| `moduleDetection: "force"`         | すべてのファイルをモジュールとして扱い、暗黙スクリプトを排除<br> グローバル汚染や読み込み順依存を避け、大規模構成での挙動を安定化                                |
| `resolveJsonModule`                | `*.json` を型付き import 可能にして設定/定数の取り回しを改善<br> JSON の構造に補完が効き、設定の互換性チェックがしやすい                         |
| `skipLibCheck`                     | 外部 `.d.ts` の再検査を省き、外部依存の型ズレで止まりにくくする<br> 依存更新に強く、CI/ローカルの型チェックも軽量化（厳密性はやや下げる）                      |
| `verbatimModuleSyntax`             | import/export の書き換えを最小化して意味を保持<br> ツリーシェイクの結果が安定し、最終成果物が軽量化しやすい（バンドラ互換性↑）                          |
| `forceConsistentCasingInFileNames` | ファイル名の大文字小文字ゆれをエラー化<br> mac では動くのに CI(Linux) で壊れる、といったクロスOS不整合を事前に回避                               |

### next.json（Nextアプリ）

#### 汎用設定

| 設定                                     | 解説                                                                                                                                     |
|:---------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------|
| `jsx: "preserve"`                      | JSX を TS が変換せずそのまま残し、後段（SWC/webpack）が処理できるようにする<br> `use client` の境界判定や Server/Client Component の最適化を Next に委譲でき、RSC 変換と整合が取れる         |
| `lib: ["ES2022","DOM","DOM.Iterable"]` | ブラウザ API（DOM/Iterable）とモダン標準 API を型として有効化<br> Client Component で DOM 補完が利く一方、Server 側で DOM を参照すると型で気づける（境界違反の早期発見）                     |
| `moduleResolution: "Bundler"`          | パッケージ解決をバンドラ挙動に合わせる（`exports` 条件・サブパス・エイリアスなど）<br> 型解決と実行時解決が一致し、「型だけ赤い/実行は通る」ズレを解消。`next/*` 仮想モジュールとも相性良い                             |
| `plugins: [{ "name": "next" }]`        | Next.js 用 TS プラグインでフレームワーク固有の検査を強化<br> `generateMetadata` / `generateStaticParams` / Route Handlers などの型誤用を早期に検出できる                    |

#### 個別設定

| 設定                                                                         | 解説                                                                                                                                                                                                                                                     |
|:---------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `include: ["**/*.ts", "**/*.tsx", "next-env.d.ts", ".next/types/**/*.ts"]` | 型チェック対象を TS/TSX ファイルと Next.js が自動生成する型ファイルに限定<br> <br>・`next-env.d.ts`: Next.js 基本型（`NextPage` や `next/image` 等）を有効化。外すと Next 固有の型がエラーになる<br>・`.next/types/**/*.ts`: App Router 専用に生成されるルート型を取り込み。外すと `params` や `generateMetadata` が `any` になり型安全性を失う |

### browser.json（React UIライブラリ）

#### 汎用設定

| 設定                                    | 解説                                                                                                                     |
|:--------------------------------------|:-----------------------------------------------------------------------------------------------------------------------|
| `jsx: "react-jsx"`                    | TypeScript に新しい React JSX 変換を使わせる設定<br> `React.createElement` を書かずにコンパクトなコードを扱える。UIライブラリとして利用される側でも、JSX の型推論が正しく効く     |
| `types: ["react","react-dom","node"]` | グローバルに読み込む型定義を指定<br> `react`/`react-dom` の型で JSX/DOM の型付けが安定。Node 型は UI 専用なら不要だが、SSRやStorybookでの Node API 利用を想定するなら役立つ |


## tsup

ビルド＆バンドルに利用する。

### 共通設定

| 設定                | 解説                                                                    |
|:------------------|:----------------------------------------------------------------------|
| `treeshake: true` | すべてのビルドでツリーシェイクを有効化<br> 未使用コードを落としてバンドルを軽量化UI/Functions/Libraryすべてに有効 |

### `browserPreset`（React UIライブラリ）

| 設定                                            | 解説                                                             |
|:----------------------------------------------|:---------------------------------------------------------------|
| `format: ["esm"]`                             | ESM のみ<br> モダンバンドラ/Next 環境は ESM 前提。CJS は不要かつ複雑化要因              |
| `platform: "browser"`                         | ブラウザ実行を前提に解決/ポリフィルを最適化<br> DOM 依存や Web API 前提コードが混在しうる UI 層に適合 |
| `target: "es2022"`                            | モダンブラウザ向け出力<br> Next/ESBuild の既定と親和性高く、不要なダウンレベル化を避ける          |
| `splitting: true`                             | コード分割を許可<br> 複数エントリ/動的 import でのキャッシュ・再利用・初期ロード削減に有利           |
| `sourcemap: false`                            | 既定ではオフ<br> 最終アプリ側のビルド/デバッグフローと二重になりがち。必要時のみON                  |
| `minify: false`                               | 既定は可読性優先<br> Next 本体側で最終最適化されるため、ここでは縮小必須でない                   |
| `external: ["react","react-dom","next", ...]` | React/Next を確実に external<br> ダブルバンドル回避とホストアプリ（Next）側の最適化を尊重    |

#### ビルド

Client Componentの成果物には、先頭に`"use client";`が必要。
しかし、`tsup`等でビルドすると、tsxファイル先頭の`"use client";`は削除されてしまう。
対策として、ビルド完了後に成果物の先頭に`"use client";`を後から付与する。

##### ポイント
- ClientサイドのコードとSeverサイドのコードを別々にバレルする
- 成果物はClientサイドとServerサイドのそれぞれでバンドルする
- `tsup`の`onSuccess`フックで、Clientサイドの成果物の先頭に`"use client";`を付与する

※コンポーネント毎にビルドし、それぞれに`"use client";`を付与する方法もあるが、管理が少々煩雑になる

##### 設定

```js
// tsup.client.config.mjs
export default await browserPreset({
  bundle: true,
  entry: { index: "src/index.client.ts" },
  onSuccess: "node ./scripts/ensure-use-client.mjs dist/client/index.js",
  outDir: "dist/client",
  splitting: false,
});
```

```js
// tsup.server.config.mjs
export default await browserPreset({
  bundle: true,
  entry: { index: "src/index.server.ts" },
  outDir: "dist/server",
  splitting: false,
});
```

### functionsPreset（Firebase Functions）

| 設定項目               | 解説                                                           |
|:-------------------|:-------------------------------------------------------------|
| `format: ["esm"]`  | ESM 出力<br> Node 18+／22 環境での ESM サポートに合わせ、将来性と互換性を確保          |
| `platform: "node"` | Node 向け解決・ポリフィル<br> GCF/Functions の Node 実行に適合               |
| `target: "node22"` | Functions の実行環境（Node 22）に最適化<br> 余計なトランスパイルを避け、起動速度/互換性を担保   |
| `splitting: false` | コード分割しない<br> 単一ファイルのほうがデプロイ/コールドスタートの挙動が読みやすく、運用が安定          |
| `sourcemap: true`  | 常時ソースマップ<br> 本番障害解析（Stacktrace 解決）が重要Functions では運用保守性を優先    |
| `minify: isProd`   | 本番/CI では縮小、それ以外は可読性優先<br> ランタイム転送量/起動時間を抑えつつ、ローカル開発はデバッグ性を維持 |
| `dts: false`       | 型定義は配布不要<br> 実行用アプリケーション配布（＝ライブラリ配布ではない）ため                   |

#### ビルド

Firebaseの仕様から、ビルドまわり（特に依存管理）が少々特殊になっている。
詳細は[こちらの投稿](https://blog.o3osatoshi.engr.work/tech/arch/firebase-monorepo-with-turborepo)を参照のこと。

### publicDualPreset（外部配布ライブラリ）

| 設定項目                                      | 解説                                                                  |
|:------------------------------------------|:--------------------------------------------------------------------|
| `format: ["esm","cjs"]`                   | **デュアル出力**<br> 依然として CJS を要求するツール/環境に配慮しつつ、モダン環境の ESM も提供採用障壁を下げる   |
| `platform: "node"`<br> `target: "es2022"` | ライブラリ出力としてモダン Node/バンドラで扱いやすい設定<br> 余計なダウンレベル化を避けつつ、幅広いツールチェーンで安定動作 |
| `splitting: true`                         | コード分割許可<br> 複数エントリや副次的モジュールを持つ配布に対応消費側のバンドラにとっても再利用性が高い             |
| `sourcemap: isProd`                       | 本番ビルドでソースマップを同梱<br> 利用者のデバッグ体験向上開発時は不要、配布時のみ付けるバランス                 |
| `minify: false`                           | 既定は非縮小<br> 配布ライブラリは利用側で最終最適化されることが多く、可読性・差分レビューを重視必要に応じて ON 可       |


## biome

フォーマット＆リントに利用する。

### base.json

| 設定                                     | 解説                                                                       |
|:---------------------------------------|:-------------------------------------------------------------------------|
| assist.actions.source.organizeImports  | インポート整理の自動修正をオフ<br>ESLintの `perfectionist` を優先                           |
| linter.domains.project                 | Project ドメインの推奨ルールを有効化<br>依存関係や import サイクルなどを検知                         |
| linter.domains.test                    | Test ドメインの推奨ルールを有効化<br>テストコードの品質と誤用防止に寄与                                 |
| linter.rules.complexity.useLiteralKeys | リテラルキー強制をオフ<br>TS の `noPropertyAccessFromIndexSignature` を優先し、ブラケット記法を許容 |

#### 適用

全体にbiomeを反映させるため、ルートに以下の`biome.json`を配置する。

```json5
// /biome.json
{
  "extends": ["@o3osatoshi/config/biome/base.json"],
  "root": true
}
```

### next.json（Nextアプリ）

| 設定                  | 解説                                                            |
|:--------------------|:--------------------------------------------------------------|
| linter.domains.next | Next.js ドメインの推奨ルールを有効化<br>Next.js 固有のアンチパターン防止やベストプラクティスを自動検知 |

#### 適用

個別に`biome.json`を配置することで、設定の上書きが可能。nextやreact用の設定反映範囲を限定できる。

```json5
// /apps/web/biome.json
{
  "extends": [
    "@o3osatoshi/config/biome/base.json",
    "@o3osatoshi/config/biome/react.json",
    "@o3osatoshi/config/biome/next.json"
  ],
  "root": false
}
```

### read.json（Reactアプリ）

| 設定                                          | 解説                                                                        |
|:--------------------------------------------|:--------------------------------------------------------------------------|
| linter.domains.react                        | React ドメインの推奨ルールを有効化<br>Hooks や JSX に関するベストプラクティスを自動検知                    |
| linter.rules.nursery.useSortedClasses       | JSX の className 等をソート<br>`clsx`, `cva`, `tw` 系関数や `classList` 属性のクラス順序を強制 |
| linter.rules.suspicious.noDuplicateJsxProps | JSX 属性の重複を禁止<br>予期しない上書きやバグを防止                                            |

#### 適用

```json5
// /packages/ui/biome.json
{
  "extends": [
    "@o3osatoshi/config/biome/base.json",
    "@o3osatoshi/config/biome/react.json"
  ],
  "root": false
}
```

## eslint

ソートのみに利用する。
biomeのカバー範囲がまだまだ狭いため、補完的にeslintを利用する。

### config

ライブラリ毎にconfigを作成し、一つにまとめてexportする。

```js
import jsonc from "./jsonc.mjs";
import perfectionist from "./perfectionist.mjs";
import vitest from "./vitest.mjs";

export default [
  {
    ignores: [
      "**/node_modules/**",
      "**/dist/**",
      "**/generated/**",
      "**/.next/**",
      "**/.turbo/**",
      "**/.idea/**",
      "**/storybook-static/**",
    ],
  },
  ...perfectionist,
  ...jsonc,
  ...vitest,
];
```

### 適用

プロジェクトルートでは、importしたconfigをそのままexportする。

```js
// /eslint.config.mjs
import config from "@o3osatoshi/config/eslint/config.mjs";

export default [...config];
```
