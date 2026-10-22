---
layout:        post
title:         "Turborepoで構築するFirebaseモノレポ環境"
date:          2025-08-13 00:00:00 +0900
modified_date: 2025-08-13 00:00:00 +0900
categories:    [tech, arch]
tags:          [coding]
---

![thumbnail](/assets/2025-08-13-firebase-monorepo-with-turborepo/thumbnail.png)

{% include coding-notice.md %}

## 前提

- functionsのみのモノレポ化を対象としている
- hostingはモノレポ化の対象外（当方はvercelを利用している）

## デプロイフロー

### Firebase Functionsモノレポの問題点

#### Firebase Functionsのデプロイフロー（仕様）

1. 対象ディレクトリ（ `apps/functions` ）をサーバにアップロード
2. アップロードされたディレクトリで `npm install` を実行

#### 問題点
`npm install` 時に、ローカルパッケージ（ `packages/application`等 ）を見つけられずにエラーになる。

### 解決策

#### バンドルを組み合わせたデプロイフロー

1. functionsのビルド＆バンドルをデプロイ前に行う
2. 対象ディレクトリ（ `apps/functions` ）をサーバにアップロード
3. アップロードされたディレクトリで `npm install` を実行

#### ポイント
ローカルパッケージへの依存をバンドルにより事前に解決する。


## 構成

```text
⏺ root/
  ├── 📁 apps/
  │   ├── 📁 web/                          # Next.js
  │   │
  │   └── 📁 functions/                    # Firebase Cloud Functions
  │       ├── 📁 src/
  │       │   └── 📄 index.ts
  │       ├── 📄 package.json
  │       ├── 📄 tsconfig.json
  │       ├── 📄 tsup.config.ts
  │       └── 📄 turbo.json
  │
  ├── 📁 packages/
  │   ├── 📁 application/                  # アプリケーション層
  │   │   ├── 📁 src/
  │   │   │   └── 📄 index.ts
  │   │   ├── 📄 package.json
  │   │   └── 📄 tsconfig.json
  │   ├── 📁 domain/                       # ドメイン層
  │   │   ├── 📁 src/
  │   │   │   └── 📄 index.ts
  │   │   ├── 📄 package.json
  │   │   └── 📄 tsconfig.json
  │   └── 📁 prisma/                       # インフラストラクチャ層
  │       ├── 📁 src/
  │       │   └── 📄 index.ts
  │       ├── 📄 package.json
  │       └── 📄 tsconfig.json
  │
  ├── 📄 package.json                      # ワークスペース設定
  ├── 📄 turbo.json                        # Turborepo設定
  └── 📄 firebase.json                     # Firebase設定
```


## 設定
モノレポ化にあたって、重要なポイントのみ説明する。
フルコードは、[GitHubリポジトリ](https://github.com/o3osatoshi/portfolio)を参照のこと。

### apps/functions

#### functions/src/index.ts

1. 通常通りにローカルパッケージからインポートする

```typescript
import { GetTransactionsUseCase } from "@repo/application"; // <------------- 1
import { PrismaTransactionRepository } from "@repo/prisma"; // <------------- 1
import { onRequest } from "firebase-functions/v2/https";

const repo = new PrismaTransactionRepository();
const usecase = new GetTransactionsUseCase(repo);

export const getTransactions = onRequest(async (request, response) => {
  const transactions = await usecase.execute();
  response.json(transactions);
});
````

#### functions/package.json

1. `tsup` でビルド＆バンドルを行う
2. ローカルパッケージへの依存を明記しない

```json5
{
  "scripts": {
    "build": "tsup" // <----------------------------------------------------- 1
  },
  "dependencies": { // <----------------------------------------------------- 2
    "firebase-admin": "^12.6.0",
    "firebase-functions": "^6.0.1"
  },
  "devDependencies": { // <-------------------------------------------------- 2
    "firebase-functions-test": "^3.1.0",
    "tsup": "^8.4.0",
    "typescript": "^4.9.0"
  }
}
```

applicationパッケージ等への依存を明記すると、Firebase側のインストールでエラーが発生してしまう。
ローカルビルド時にバンドルすることでエラーを避ける。

#### functions/tsconfig.json

1. applicationパッケージ等への依存はパスエイリアスで解決する

```json5
{
  "compilerOptions": {
    "paths": {
      "@repo/application": ["../../packages/application/src"], // <---------- 1
      "@repo/prisma": ["../../packages/prisma/src"] // <--------------------- 1
    }
  }
}
```

#### functions/tsup.config.ts

1. バンドルサイズ削減とビルド短縮のため基本的に除外し、Firebase側のインストールに任せる  
   ※ `autoExternals` でpackage.jsonの `dependencies` と `devDependencies` を配列化する

```typescript
export async function functionsPreset(opts: Options = {}) {
  return defineConfig({
    external: [ // <--------------------------------------------------------- 1
      ...autoExternals(),
      ...((opts.external ?? []) as NonNullable<Options["external"]>),
    ],
    format: opts.format ?? ["cjs"],
  });
}
```

#### functions/turbo.json

1. ルートの `turbo.json` を継承する
2. application等への依存を明示する

```json5
{
  "extends": ["//"], // <---------------------------------------------------- 1
  "tasks": {
    "build": {
      "dependsOn": ["@repo/application#build", "@repo/prisma#build"] // <---- 2
    }
  }
}
```

Turborepoは `package.json` の `dependencies` と `devDependencies` に記載された情報から依存関係を自動検知する。
今回の場合、 `package.json` でapplicationパッケージ等への依存を明記していないため、Turborepoには依存関係が認識されない。
この状態で `turbo run build --filter=functions...` を実行しても、applicationパッケージ等のビルドは行われない。
そこで `turbo.json` にapplicationパッケージ等への依存を明示することで、functionsのビルド前にapplicationパッケージ等のビルドを強要する。

### packages/application
Firebaseを意識した特別な設定はなし。

#### application/src/index.ts

```typescript
export class GetTransactionsUseCase {
  constructor(private readonly repo: TransactionRepository) {}

  execute(): ResultAsync<GetTransactionsResponse, Error> {
    return this.repo.findMany().map(toTransactionsResponse);
  }
}
```

#### application/package.json

1. src-firstにexportsを設定する  
   ※ dist-firstでも問題はない

```json5
{
  "exports": {
    ".": "./src/index.ts" // <----------------------------------------------- 1
  },
}
```

### packages/prisma
Firebaseを意識した特別な設定はなし。

#### prisma/schema/schema.prism
v16.16.0からrustエンジンの利用を避けられる（詳細は[こちら](https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/no-rust-engine)）。
rustエンジンは往々にしてデプロイ関連でエラーを引き起こすため使用を避ける。

```prisma
generator client {
  provider   = "prisma-client"
  output     = "../generated/prisma"
  engineType = "client"
}
```

#### prisma/src/index.ts

```typescript
export class PrismaTransactionRepository implements TransactionRepository {
  findMany(): ResultAsync<Transaction[], Error> {
    return ResultAsync.fromPromise(
      prisma.transaction.findMany(),
      (e) =>
        newPrismaError({
          action: "FindTransactions",
          cause: e,
        }),
    ).andThen((rows) => Result.combine(rows.map(toEntity)));
  }
}
```

### root

#### package.json
Firebaseを意識した特別な設定はなし。

```json5
{
  "scripts": {
    "build:functions": "turbo run build --filter=@o3osatoshi/functions...",
    "deploy:functions": "firebase deploy --only functions"
  }
}
```

#### turbo.json
Firebaseを意識した特別な設定はなし。

```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": ["$TURBO_DEFAULT$", ".env*"],
      "outputs": ["dist/**"]
    }
  }
}
```

#### firebase.json

1. rootからの相対パスでfunctionsを指定する
2. デプロイ前にfunctionsのビルド（ `pnpm build:functions` ）を実行する

```json5
{
  "functions": [
    {
      "source": "apps/functions", // <--------------------------------------- 1
      "predeploy": "pnpm build:functions" // <------------------------------- 2
    }
  ]
}
```


## メリット・デメリット

### メリット
- Firebaseの問題をfunctionsディレクトリ内で完結して解消している
   - applicationやrootの設定・コードに変更が不要（汚染されない）
- バンドルの設定がシンプル
   - 複雑な設定・構成を必要としない
- ソースコード（ `functions/src/index.ts` ）内でローカルパッケージをインポートできる
   - ローカルパッケージへのジャンプも機能し、開発体験を損なわない（その逆も可能）

### デメリット
- 今のところ見つかっていない
