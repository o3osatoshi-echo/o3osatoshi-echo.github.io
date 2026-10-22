---
layout:        post
title:         "実践 Next.js"
date:          2025-04-05 00:00:00 +0900
modified_date: 2025-04-05 00:00:00 +0900
categories:    [tech, front]
tags:          [reading]
---

![thumbnail](/assets/2025-04-05-jissen-nextjs/thumbnail.png)

{% include reading-notice.md %}

## Next.jsの基礎

#### ナビゲーション

##### ハードナビゲーション
ブラウザが画面を再読込する画面遷移。

##### ソフトナビゲーション
ブラウザが画面を**再読み込みしない**画面遷移。
必要な箇所だけ再レンダリングする。
一度ブラウザに確保した値、レンダリングした画面をキャッシュし、画面をリロードするまで利用する。


## Server Componentとレンダリング

#### Server Component
サーバサイドでのみ実行される。ブラウザに実行すべき**JSが送られない**。

「Client Componentでしか使えないAPIが存在します」**エラーが出たら**、`use client`を宣言する。
エラーが出るまでは宣言せず、必要最低限にとどめる。

Client ComponentにimportされるとClient Componentになる。

##### Client Componentを使うべきケース
- インタラクティブな機能を持つ
- コンポーネントに保持した状態を扱う

##### データ取得
セキュリティとパフォーマンスの観点から、可能な限り**Server Componentでデータを取得する**。

#### レンダリング

##### 種別

- 静的レンダリングRoute
  - すべてのリクエストに同一のレンダリング結果をレスポンス
  - `◯` アイコン
- 動的レンダリングRoute
  - **リクエストごとに異なるレンダリング結果**をレスポンス
  - `λ` アイコン

##### 動的レンダリングRouteの要因

1. **動的データ取得**の使用
   - cache不可のfetchが含まれる
2. **動的関数**の使用
   - 動的関数とは、HTTPリクエストの内容を参照する関数のこと（cookiesなど）
3. **Dynamic Segment**の使用


## App Routerの規約

#### Segment構成ファイル

| ファイル           | 補足                                  |
|:---------------|:------------------------------------|
| React Suspense | 子の読み込み完了までフォールバックUIを表示する            |
| Error Boundary | 子がErrorをスローした際にフォールバックUIを表示する       |
| not-found      | `notFound()`が実行された際に表示する            |
| error          | layoutはerrorの親階層に位置する               |
| global-error   | error.tsxでErrorがハンドリングされなかった際に表示される |

#### Segment構成フォルダ

- Route Groups  
「用途」ごとに異なるLayoutを適用したい場合に活用できる。
- Private Folder  
`_`を接頭辞にもつフォルダのこと。
特定機能の関連ファイルをまとめる（コロケーション）際に活用できる。

#### Parallel RoutesとIntercepting Routes

##### Parallel Routesの概要
`@xx`をRootとしたSubtreeと、表示しているRoute Segmentが一致したタイミングで表示される。
一度表示されたあとは、一致しないRoute Segmentにソフトナビゲーションしても表示され続ける。

##### Intercepting Routesの概要
Routeを横取りする（インターセプトする）。

#### Routeのメタデータ
祖先Segmentのメタデータを継承する。

- 静的メタデータ ... `metadata`オブジェクトをexportする
- 動的メタデータ ... `generateMetadata`関数（非同期関数）をexportする


## データ取得とキャッシュ

#### fetch関数のRequestのメモ化

```react
export default function Page({ params, searchParams }: Props) {
  const categoryName = params.categoryName;
  const page = typeof searchParams.page === "string" ? searchParams.page : "1";
  const data = await getCategory({ categoryName, page, take: "15" });
}
```

パラメータが異なる場合、「同一のfetch関数リクエスト」とみなされない。
パラメータ差分が発生しないようgetCategory関数を直接使用せず、共通関数に処理をまとめる。

#### fetch関数のキャッシュ
キャッシュを活用すると、WebAPIサーバへのアクセス頻度を削減できる。
また、データ取得の遅延が削減されるため、画面を提供する速度が向上する。

next.revalidateオプションは、キャッシュの有効期間を秒数で指定するオプション。

#### Prisma Clientでのデータ取得
ネストしたオブジェクトを渡すことで、リレーションフィールド値を取得できる。

```typescript
const users = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    image: true,
    profile: { select: { screenName: true } },
  },
});
```

#### cache関数を使用したRequestのメモ化

cache関数を使用するときの注意点
- Reactコンポーネントの中で使わない
- データ取得関数の引数はプリミティブ値でなければならない


## 認証機能

#### Middleware
リクエスト/レスポンスのヘッダー書き換えやリダイレクトといった**中間処理**を行える。
NextAuth.jsでは、特定のURLパターンにマッチするRouteを「認証が必要なRoute」としてバンドリングする。

#### getServerSession
サーバサイド処理でログインユーザ情報を参照できる。
以下のようにすれば、都度authOptionsをimportせずに済む。

```typescript
import { getServerSession as originalGetServerSession } from "next-auth";

export const getServerSession = async () => {
  return originalGetServerSession(authOptions);
};
```

#### 呼ばれることのない関数と型推論
Middlewareで事前チェックが行われるため、Page内で実際にnotFound関数が呼ばれることはない。
ただ、if文により型推論の結果が変化し、後続のコードを安全に書くことができる。

```typescript
const session = await getServerSession();
if (!session || !session.user) notFound();
```


## Parallel RoutesとIntercepting Routesを用いたモーダル

##### メリット

- 遷移元の画面に戻らずにモーダル上で次々と画面遷移できる
- モーダル（遷移先）のURLをシェアできる

#### フォルダ構成

```
/app
├── @modal
│   ├── default.tsx
│   └── (.)photos
│       └── [id]
│           └── page.tsx
├── photos
│   └── [id]
│       └── page.tsx
```

ソフトナビゲーション時に限り`(.)photos`配下の画面を提供するようになる。
欠かせないのが`default.tsx`。**お決まり**のようにnullをreturnする。

##### モーダルの実装
`@modal/(.)photos/[id]/page.tsx`は、UIとしては画面を覆うモーダルだが、Pageファイルとしての実装になる。
**モーダル画面であるにもかかわらず、Server Componentsならではの処理が可能**。

## データ更新とUI

#### Server Actionの概要
Formからサーバの非同期関数を直接呼び出せる機能のこと。

#### pendingステート

```typescript
const { pending } = useFormStatus();
```

form要素の子コンポーネントで使用できる。

#### 強制再レンダリング
keyの指定により、強制的に再レンダリングを行える。

```react
<AlertDialogModalComponent
  key={state.updatedAt} // エラーが発生するごとに再マウント、内部状態が破棄される
  status={state.error.status}
/>
```

#### Server ActionのFormバリデーション
Progressive Enhancementを維持するため、actionとonSubmitを分ける。
**onSubmit時にバリデーションを行う**。

```react
<form action={formDispatch} onSubmit={handleSubmit}>
```

Server Actionを使用したFormにおいて、クライアントサイドのバリデーションをどのように行うかは、現時点でベストプラクティスが確立されていない。

### Revalidation

#### バリデート識別子

- revalidatePath ... 特定のRouteのパスでキャッシュを無効化する
- revalidateTag ... 特定のタグ文字列でキャッシュを無効化する

##### revalidateTagの設計
revalidateTagは複数指定できる。抽象的と具体的の両方を指定することで、状況に応じてrevalidateTagを実行できる。

- **抽象的**なタグ ... 無効化が楽
- **具体的**なタグ ... データソースアクセス効率が良い

#### On-demand Revalidation
**特別な理由がない限り**「データの作成、更新、削除」には、Route Handlerではなく**Server Actionを使用する**。
Server Actionは直接サーバサイドの関数を呼べるだけでなく、**On-demand Revalidationに優位性**がある。

##### Revalidationの種類

- Time-based Revalidation ... キャッシュの**有効期限**を指定する
- On-demand Revalidation ... キャッシュを**任意**のタイミング（データ更新直後等）で無効化する

##### Route Handlerの課題
router.refresh()の併用が必須。
router.refresh()は、「Routerキャッシュの削除」と「現在Routeの再レンダリング」をトリガーする。
データ更新直後には別画面への遷移をともなうため、再レンダリング（リクエスト）が余分になる。

##### Server Actionの利点
router.refresh()を実行しなくても、「Routerキャッシュの削除」が行われる。
処理の最後にredirect関数を使用して遷移すれば、更新と同時に画面遷移を行える。

### Errorのハンドリング
**エラーを状態として管理**する。

```typescript
export type FormState = {
    updatedAt: string;
    liked: boolean;
    likedCount: number;
    error: { message: string; status: number } | null;
};

export const initialFormState = (
    initialState?: Partial<FormState>
): FormState => ({
    updatedAt: Date.now().toString(),
    liked: false,
    likedCount: 0,
    error: null,
    ...initialState,
});
```

```typescript
try {
    const {imageUrl, name, screenName, bio} = validateFormData(formData);
    const userId = session.user.id;
    await prisma.user.update({
        where: {id: userId},
        data: {name, image: imageUrl},
    });
    await prisma.profile.update({
        where: {userId},
        data: {screenName, bio, userId},
    });
    revalidateTag(`users/${userId}`);
} catch (err) {
  if (err instanceof PrismaClientKnownRequestError) {
    if (err.code === "P2002") {
      return { message: "「表示名」がすでにしようされています" };
    }
  }
  return { message: "エラーが発生しました" };
}
redirect("/profile");
```

```typescript
export const handleSuccess = (prevState: FormState): FormState => ({
    ...prevState,
    updatedAt: Date.now().toString(),
    liked: true,
    likedCount: prevState.likedCount + 1,
    error: null,
});

export const handleError = (
    prevState: FormState,
    error: { message: string; status: number }
): FormState => ({
    ...prevState,
    updatedAt: Date.now().toString(),
    error,
});
```


## パフォーマンスとキャッシュ

#### コンポーネント構造のパフォーマンスへの影響

##### コロケーション
コンポーネントが必要とするデータをその**コンポーネント自身で取得する**こと。これにより**パフォーマンスが改善**する。

##### 動的関数と動的データ取得
動的関数が使われて以降のfetchが「動的データ取得」として扱われる。
しかし、動的関数が使われたからといって、データ取得のすべてが「動的データ取得」でなければならないとは限らない。

#### 静的Route
**静的Routeのみを含むレイアウトが望ましい**。
UIに必要な情報のみCSR（Client-side Rendering）で取得するように修正する。
コンポーネントツリーのRoot付近で動的関数を使用する際には特に注意が必要。

ただし、すべての状況において静的Routeが望ましいわけではない。
CSRはSSRと比較してネットワークリクエストが多くなる。
表示する画面の大部分が動的データであるなら、SSRを選んだほうがパフォーマンス指標は良くなる。

### SSG

#### Dynamic RouteのSSG
「idが1の場合にはこのレスポンス、2の場合にはこのレスポンス」のように、事前に準備する。
generateStaticParams関数は、ビルド時にしか実行されない関数で、事前に生成するDynamic Routeを指定する。

searchParamsを参照するとRouteが「動的Route」になってしまい「SSG Route化」できない。
urlパラメータではなく、pathパラメータを利用する。

#### パフォーマンス
SSG Routeへのリクエストは、リクエスト時のサーバ処理が省略されるため、レスポンスが高速化される。
TTFBの遅れは多くのパフォーマンス指標に影響を与える。
可能なら、できるだけSSG Route（静的Route）にする。

- TTFB (Time To First Byte) ... サーバからブラウザに最初のデータが届くまでの時間
- LCP (Large Contentful Paint) ... メインコンテンツが表示されるまでの読み込み時間

### アセット最適化
画像はページのパフォーマンスを決める重要な要素。画像配信の最適化はパフォーマンス向上に必須。

##### 画像サイズの指定
Layout Shift（画像が読み込まれるときにほかの要素を動かしてしまう）は、パフォーマンスを低下させる。

##### priority props
LCPにpriorityを追加すると、画像取得の優先度があがり、LCPが向上する。

##### 必須のalt属性
装飾目的の画像では、alt属性は空文字にする。

##### 画像ホストの指定
画像の読み込みにはセキュリティリスクが伴う。ホストは絞るべき。

### 4種類のキャッシュ

1. Requestのメモ化
   - fetch関数の戻り値
   - 有効期限は「単一のブラウザリクエストが存続している間」に限られる
2. Dataキャッシュ
   - fetch関数で取得したデータ
   - 誰とでも共有可能な「静的データ」に限られる
3. Full Routeキャッシュ
   - HTML、RSC Payload
   - 異なるユーザ間で共有されるため「静的レンダリングRoute」に限られる
4. Routerキャッシュ
   - RSC Payload


## Prisma

##### マイグレーション
「テーブルの作成・初期化・関連付け」などを行うDBの初期化と更新プロセスのこと。

##### シーディング
DBのテーブルに初期データ（テストデータ）を投入する作業のこと。
