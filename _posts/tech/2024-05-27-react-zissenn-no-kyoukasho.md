---
layout:        post
title:         "React実践の教科書"
date:          2024-05-27 00:00:00 +0900
modified_date: 2024-05-27 00:00:00 +0900
categories:    [tech, front]
tags:          [reading]
---

![thumbnail](/assets/2024-05-27-react-zissenn-no-kyoukasho/thumbnail.png)

{% include reading-notice.md %}

## JSの予備知識

#### DOM
Document Object Model。HTMLを解釈し、木構造で表現したもの。

##### JSによる手続き的なDOM操作
従来のJSでは、**DOMを直接指定して画面の要素を書き換え**ていた。
コードが肥大化してくると、表示速度に問題が生じたり、どこで何をしているのかわからなくなったりした。

##### 仮想DOM
**JSのオブジェクトで作られた**仮想的なDOMのこと。

#### パッケージマネージャー
パッケージの管理、インストール、アップグレート等をになってくれる管理ツール。

##### 近代JSの転換期
**ES2015(ES6)**で、ECMAScriptの**大きな改定**が行われた。
let・constを用いた変数宣言、アローファンクション、Class構文、分割代入、テンプレート文字列、スプレッド構文、Promise等が追加された。

##### モジュールバンドラー
「開発はファイルを分け、本番は１ファイルにまとめる」という思想を実現するために、
JSファイルやCSSファイル等をまとめてくれるモジュールバンドラーが作られた。
現在の主流はwebpack。

##### トランスパイル
古いブラウザのために、JSの記法を古い記法に変換してくれる。
現在の主流はBabel。

##### 開発体験の優れたVite
開発環境では、ソースコードのバンドルを行わないため、高速に実行できる。

#### SPA
HTMLファイルは１つで、JSでDOMを書き換えることで画面遷移を実現する。
そのため、従来のWebシステムのようにページ遷移の際に画面が一瞬白くなる（ちらつく）ことがない。

##### ユーザ体験の向上
ページの表示速度は重要で、特にECサイト等では売上に直結する。

> 表示速度が0.1秒遅くなれば1%売上が減少し、1秒早くなれば10%売上が増加する

##### フロントエンドエンジニアとは
フロントに特化しつつ、バックエンドやDBの知識もつけていかないといけない。

- バックエンドも一気通貫に対応できるほうがチームは助かる
- アプリケーションを設計するためには一貫した知識が必要


## JSの基礎
ざっくり概念を把握したあとは、手を動かしながら細かい知識を補完する。

#### 記法

##### const, let
オブジェクトと配列の定義には、constを使用する。

##### 配列の分割代入
```javascript
// 1つ目のみ必要な場合
const [ name ] = myProfile;
```

##### スプレッド構文
```javascript
const arr1 = [1, 2];
const summaryFunc = (num1, num2) => console.log(num1 + num2);

// 普通に配列の値を渡す場合
summaryFunc(arr1[0], arr1[1]);

// スプレッド構文を用いた方法
summaryFunc(...arr1);
```

##### mapの第2引数
mapの第2引数にはindexの情報が格納される。

```javascript
const nameArr = ["田中", "山田", "佐藤"];
nameArr.map((name, index) => console.log(`${index + 1}番目は${name}です`));
```

##### 三項演算子
**判定 → 変数設定**に有用。

##### 論理演算子の意味
`||` は、左側がfalse判定なら右側を返す。
`&&` は、左側がtrue判定なら右側を返す。

#### DOMアクセス
`document` はDOMツリーのエントリーポイント。

```javascript
const containers = document.getElementsByClassName("container");
```

##### jQuery
jQueryはJSライブラリで、DOMアクセスを簡単に扱えるようにしてくれるもの。
`document.getElementById("nushida")`は、`$("#nushida")`と書ける。


## Reactの基礎

##### Propsの扱い方
Propsをどう扱うか（destructureするかいなか）は、プロジェクト毎にルールを決めて統一する。

```jsx
// 分割代入しない
const Profile = (props) => {
  return (
    <div>
      <h1>{props.name}</h1>
      <p>{props.age}</p>
    </div>
  );
};

// 分割代入
const Profile = (props) => {
  const { name, age } = props;
  return (
    <div>
      <h1>{name}</h1>
      <p>{age}</p>
    </div>
  );
};

// 引数で分割代入
const Profile = ({ name, age }) => {
  return (
    <div>
      <h1>{name}</h1>
      <p>{age}</p>
    </div>
  );
};
```

##### useStateの更新関数
set関数を2行続けて書けば違いが分かる。

```jsx
// 厳密に正しくない
setNum(num + 1);

// 関数内で関数を指定
setNum((prev) => prev + 1);
```

## ReactのCSS

#### CSS Modules
React開発ではコンポーネント毎にCSSファイルを用意することが多い。

##### CssModules.modules.scss
```scss
.container {
  background-color: #f0f0f0;
}
.title {
  font-size: 20px;
}
```

##### CssModules.jsx
```jsx
import styles from "./CssModules.module.scss";

export const CssModules = () => {
  return (
    <div className={styles.container}>
      <p className={styles.title}>- CSS Modules -</p>
    </div>
  );
};
```

#### Styled JSX
コンポーネントファイルにCSSを記述する（CSS-in-JS）ライブラリ。
SCSS機能は、別途設定等を行わない限り使用できない。
積極的に採用しているチームは少ない。

##### StyledJsx.jsx
```jsx
export const StyledJsx = () => {
  return (
    <>
      <div className={styles.container}>
        <p className={styles.title}>- CSS Modules -</p>
      </div>

      <style jsx>{`
        .container {
          background-color: #f0f0f0;
        }
        .title {
          font-size: 20px;
        }
      `}</style>
    </>
  );
};
```

#### styled components
根強い人気を誇るCSS-in-JSライブラリ。
**SCSS記法がそのまま使え**、既存のCSSファイルからの移行がスムーズに行える。

##### StyledComponents.jsx
```jsx
import styled from "styled-components";

export const StyledComponents = () => {
  return (
    <SContainer>
      <STitle>- Styled Components -</STitle>
    </SContainer>
  );
};

const SContainer = styled.div`
  background-color: #f0f0f0;
`;
const STitle = styled.p`
  font-size: 20px;
`;
```

先頭の「S」は、「styled componentsで作成したコンポーネント」であることを示す。
プロジェクト毎に命名規則を定め、チームメンバーで統一する。

#### Emotion
styled componentsと並び根強い人気を誇るCSS-in-JSライブラリ。
これまでに紹介した３パターンの記載方法のそれぞれと似た書き方ができる。

#### Tailwind CSS
近年非常に人気なユーティリティファーストなフレームワーク。
慣れればスピードが出る。CSSをガッツリ書くデザイナーがいないチームでは選択肢の１つになる。
「命名に悩まなくて良い」ことがメリット。

#### コンポーネントライブラリ
実際の現場では、ゼロからスタイリングするのではなく、コンポーネントライブラリを使用することが多い。

- Tailwind製のHeadless UI
- Chakra UI
- Material-UI
- Semantic UI


## 再レンダリングと最適化
メモ化（処理結果の保持）により、コンポーネント・関数・変数などの再レンダリングを制御する。

#### 再レンダリング
再レンダリングとは、仮想DOMの変更を検知してコンポーネントを再処理すること。
初回レンダリング（コンポーネントのマウント）と再レンダリングは異なり、
useState等の初期値はマウント時のみ初期化される。

##### 再レンダリングのトリガー

1. **State**が更新された時
1. **Props**が変更された時
1. **親コンポーネント**が再レンダリングされた時

##### useEffect
第2引数に `[]` を指定すると、コンポーネントの初回表示時のみに実行される処理を記述できる。
初期データの取得などでよく使用される。

Stateが増えると再レンダリングも増えるため、毎回実行する必要がない処理の制御に役立つ。

#### レンダリング最適化 React.memo
**コンポーネントのメモ化**。
親コンポーネントが再レンダリングされても、PropsやStateに変更がなければ再レンダリングされない。

```jsx
const Component = memo(() => {});
```

#### レンダリング最適化 React.useCallback
**関数のメモ化**。
再レンダリングで関数は再作成されるため、関数をPropsでメモ化したコンポーネントに渡しても、Props先は再レンダリングされてしまう。
関数のメモ化で再レンダリングを抑制できる。

```javascript
const onClickReset = useCallback(() => {
    setNum(0);
}, []);
```

依存配列が空の場合、最初に作成したものが使い回される。

#### レンダリング最適化 React.useMemo
**変数のメモ化**。
変数設定のロジックが重い場合に使用することで負荷を低減できる。

```jsx
const sum = useMemo(() => {
  return 1 + 3;
}, []);
```


## グローバルなState管理

#### グローバルなState管理が必要な理由
Propsのバケツリレーによる弊害が起こるため。StateやPropsを適切に設計することは非常に重要。

- コードの複雑化（何のためのコンポーネントか分かりづらくなる）
- 再利用できなくなる
- 不必要な再レンダリングが発生する

#### ContextでのグローバルなState管理
ログインしているユーザの情報を管理する際に有用。

##### AdminFlagProvider.jsx
```jsx
import { createContext } from "react";

export const AdminFlagContext = createContext();

export const AdminFlagProvider = (props) => {
  const { children } = props;
  const [isAdmin, setIsAdmin] = useState(false);

  return (
    <AdminFlagContext.Provider value={ { isAdmin, setIsAdmin } }>
      {children}
    </AdminFlagContext.Provider>
  );
};
```

##### 再レンダリングに注意
Contextオブジェクトの値が何か１つでも更新されたとき、**そのContextを参照しているコンポーネントは全て再レンダリング**される。
１つのContextにStateをまとめて詰め込むのは避ける。

#### その他のグローバルなState管理

##### Recoil
今後間違いなく主流になる。導入と実装のハードルが低い。

##### Apollo Client
APIの取得結果とは別で任意にデータを保持できる。比較的シンプルに実装できる。
GraphQL APIのクライアントとしてApollo Clientを使用している場合に、グローバルなState管理も任せてしまうのがよい。


## カスタムフック

#### 概要
カスタムフックファイル内で**各種Hooksを使用可能**な点を除けば、**一般的な関数と同じ**。
**本来コンポーネントの責務は与えられたデータに基づいて画面の見た目を構築すること**であり、複雑なロジックはカスタムフック等に分離したほうがよい。
カスタムフックを適切に使用することで、可読性・保守性の高いReact開発が可能になる。

#### 実装

##### useMemoList.tsx
```tsx
import { useCallback, useState } from "react";

export const useMemoList = () => {
  const [memos, setMemos] = useState<string[]>([]);

  const addTodo = useCallback((text: string) => {
    const newMemos = [...memos];
    newMemos.push(text);
    setMemos(newMemos);
  }, [memos]);
  
  const deleteTodo = useCallback((index: number) => {
    const newMemos = [...memos];
    newMemos.splice(index, 1);
    setMemos(newMemos);
  }, [memos]);

  return { memos, addTodo, deleteTodo };
};
```

#### 開発・リファクタリング手順
1. まずは１つのコンポーネントに実装
1. コンポーネント化できそうな箇所を分解
1. ロジック分離できそうな箇所をカスタムフック化


## ReactとTS

##### 設定ファイル tsconfig/module
- フロントエンド ... module: esnext
- バックエンド ... module: commonjs

##### 型定義の管理方法
型定義のimport時は忘れずにtype（`import type { ~ } from`）を使用する。

##### コンポーネントの型定義
JSXを返却していないとエラーにできたり、コンポーネント独自の設定ができたりするので、
関数コンポーネントには**FCやVFCを指定**する。
