---
layout:        post
title:         "プロを目指す人のためのTypeScript入門"
date:          2024-04-14 00:00:00 +0900
modified_date: 2024-04-14 00:00:00 +0900
categories:    [tech, lang]
tags:          [reading]
---

![thumbnail](/assets/2024-04-14-Introduction-to-TypeScript-for-future-professionals/thumbnail.png)

{% include reading-notice.md %}

## イントロダクション

#### TSの特徴
- ランタイムの挙動が**型情報に依存しない**
  - TSに関数オーバーローディング（型情報をもとに関数の実体を選択する）機能がない

TSコードからJSコードへの変換はとても単純で、基本的にはTSに特有の部分**（型注釈）を取り除く**だけ。

TSでは「JSにはない"独自機能"」を追加しない方針。 enumやnamespaceは、TS初期に追加された過去の遺物。これらは複雑な挙動をするので使うべきではない。

##### 静的型付け言語のいいところ
1. 型安全性  
コンパイラが型チェック
1. ドキュメント化  
型の情報がソースコードに書かかれ、プログラム読解の助けになる

#### TSコンパイラの役割
1. 型チェック
1. トランスパイル

##### 型チェック
- 静的なチェック
  - 実際にプログラムを実行しなくても行えるチェック
- 動的なチェック
  - 実際にプログラムを実行してその結果を見てプログラムに間違いがないかを確認する


## 基本的な文法と型
### 文と式
変数宣言は、 `const 変数名 = 式;` と表される。ほかの文も式を用いて組み立てられる。
文と式の違いは、**結果**があるかどうか。

##### 識別子
ひらがな、カタカナ、漢字は識別子に使用可能。記号系は使用不可。
```typescript
const あいう = 123;
```
慣習として、変数名は小文字で始め、型名は大文字で始める。

##### constとlet
constで宣言された変数は再代入が不可能で、letで宣言された変数は再代入が可能。
letを使うと読む人に負担がかかるため、**極力const**を使うべき。

### プリミティブ型

##### 数値型
number型では整数と小数に区別がない。

##### 数値リテラル
リテラルとは、何らかの値を生み出すための式のこと。数値リテラルは式の一種。
```typescript
const binary = 0b1010; // 2進数リテラル
```

数値リテラルは数字の間に `_` を挟むことが許される。
```typescript
const million = 1_000_000;
```

##### 任意精度整数（BigInt）
BigIntリテラルは整数のあとにnを書く。
BigIntは整数しか扱えないため、除算の結果が小数になる（割り切れない）場合は整数に丸め込まれる。

##### 文字列リテラルの一種
テンプレートリテラル `Hello, world!` のようにバッククオートで囲う。

##### Unicodeコードポイント
Unicodeコードポイントを使ったエスケープシーケンス `\u{1F600}` のように書く。

##### nullとundefined
nullとundefinedでは、仕様的にundefinedのほうがサポートが手厚いため、undefinedの利用が推奨される。

#### プリミティブ型同士の変換　暗黙の型変換

型推論
: 明示的な型注釈がなくても、TSは変数の型を自動判定する

#### プリミティブ型同士の変換　明示的な変換

##### 真偽値への型変換の規則

| 型              | 真偽値への変換結果              |
|:---------------|:-----------------------|
| 数値             | 0とNaNがfalse、ほかはtrue    |
| BigInt         | 0nがfalse、ほかはtrue       |
| 文字列            | 空文字列（""）がfalse、ほかはtrue |
| null・undefined | false                  |
| オブジェクト         | true                   |

### 演算子
演算子の構成要素となっている式のことをオペランドと呼ぶ。
たとえば `x + 1` は、`x` と `1` の2つのオペランドを持つ。

##### 二項演算子
`3 + 4n` のように混ぜて使うことはできず、コンパイルエラーになる。
`-1` は、数値リテラル `1` に単項演算子 `-` がついている。

##### 単項演算子
副作用とは、返り値を返す以外に発生する影響のこと。
副作用の存在を意識させられるコードは良いコードではなく、あまり書かれない。

##### 比較演算子と等価演算子
`==` と `!=` は使うべきではない。より便密な判定を行う `===` を使用する。  
`x === null || x === undefined` と書けるが、より短い `x == null` のほうが好まれる場合もある。

##### 論理演算子 真偽値の演算
`&&` と `||` の特徴は短絡評価。演算子が左側の値を帰す場合、右側は評価すらされない。

|演算子 | xを返す条件                | yを返す条件               |
|:------|:----------------------|:---------------------|
| `x && y` | xの真偽値がfalse           | xの真偽値がtrue           |
| `x || y` | xの真偽値がtrue            | xの真偽値がfalse (デフォルト値) |
| `x ?? y` | xがnullまたはundefinedでない | xがnullまたはundefined   |

### 基本的な制御構文

##### ブロック
複数の文を1つの文にまとめる。

##### switch文
switch文の**各case節はbreak文で終わらせる**のが原則。

##### while文
break文は、while文の終了条件が式で買いきれない場合にとくに有効。


## オブジェクトの基本と型

### オブジェクトとは

#### プロパティ名の指定方法
プロパティ名には、識別子だけではなく文字列リテラルを使える。
```typescript
const obj = {
  "foo bar": -500,
};
console.log(obj["foo bar"]); // -500
```
ただし、識別子ではないプロパティ名に対しては`obj.foo`のような式でアクセスすることはできない。

#### 計算されたプロパティ名
プロパティ名を動的に決めるための構文。
```typescript
const propName = "foo";
const obj = {
  [propName]: 123,
};
console.log(obj.foo); // 123
```
オブジェクトの中身（プロパティ）を書き換えるのはconstによって制限されない。

#### スプレッド構文
既存のオブジェクトを拡張した別のオブジェクトを作る場合に有用。
スプレッド構文によって行われるのは**プロパティのコピー**であり、元のオブジェクトのプロパティを変更しても、コピー先のオブジェクトには影響しない。
スプレッド構文と通常のプロパティ宣言が同じプロパティを与える場合、**あと**に書かれているほうが採用される。

#### オブジェクトのコピー
オブジェクトは、明示的にコピーしない限り、**同一のオブジェクト**が入る。
他の場所でも同一オブジェクトを保持しており、そちらからオブジェクトが書き換えられることがある。

スプレッド構文でオブジェクトをコピーする場合、ネストしたオブジェクトは元と同一オブジェクトなので注意が必要。
ネストしたオブジェクトも含めたコピー方法に、公式的な方法は今のところない。

##### オブジェクトの同一性
オブジェクトを `===` で比較した場合、両辺が同一オブジェクトである場合に `true` になる。
中身の値を比較する方法に、公式的な方法は今のところない。

### オブジェクトの型
type文はその型を使うよりあとでもよい。 基本的に `iterface`　宣言ではなく `type` 文を使う。

#### インデックスシグネチャ
任意のプロパティ名を許容する型。
```typescript
type User = {
  [key: string]: number;
};
```

##### インデックスシグネチャに潜む罠
プロパティの存在とは無関係に、**どんな名前のプロパティにもアクセス**できてしまう。
この特性により型安全性が破壊されてしまうため、インデックスシグネチャの使用は避けるべき。
多くの場合はMapオブジェクトで代替できる。

#### 読み取り専用プロパティの宣言
プロパティの変更予定がないのなら、安全のために読み取り専用（ `readonly` ）にする。

#### typeofによる型取得
**型は設計**を表すものであり、**実装は設計に依存**して書かれる。
たまに型ではなく値が上位に来る場合がある。そういう場合にtypeofは使えるが、濫用すべきではない。

### 部分型関係

#### 部分型とは
2つの型の互換性を表す概念。型Sが型Tの部分型であるとは、S型の値がT型の値でもあることを指す。  
TSは構造的部分型。

**構造的部分型**
: プロパティを実際に比較する方式

**名前的部分型**
: 「部分型である」と明示的に宣言する方式

#### プロパティの包含関係による部分型関係の発生
```typescript
type Animal = {
  age: number;
};
type Human = {
  age: number;
  name: string;
};
```
Human型はAnimal型の部分型である。
Animal型のオブジェクトが必要な場面で、Human型のオブジェクトも使える。

### 型引数を持つ型
型を定義するときにパラメータを持たせられる。 型引数を持つ型は**ジェネリック型**と呼ばれる。

#### 型引数を持つ型の宣言
型引数は、その宣言の中でだけ有効な型名となる。
```typescript
type User<T> = {
  name: string;
  child: T;
};
```

#### ジェネリック型の使用
宣言時と同様に`<>`という記号を用いる。型引数を指定しないと型を利用できない。
```typescript
const user: User<number> = {
  name: "Taro",
  child: 123,
};
```

#### 部分型関係による型引数の制約
`extends 型` 構文は、「型引数は常に**型**の部分型でなければならない」という制約を課せる。
```typescript
type HasName = {
  name: string;
};
type Family<Parent extends HasName, Child extends Parent> = {
  parent: Parent;
  child: Child;
};
```
後続のextendsには、既出の型引数を使用できる。

#### オプショナルな型引数
型引数の指定が必須ではなくなる。デフォルト値の指定に有用。
```typescript
type Family<Parent = Animal, Child = Animal> = {
  parent: Parent;
  child: Child;
};
``` 

### 配列
配列は**オブジェクトの一種**。そのため、要素へのアクセスはプロパティアクセスと同様に行える。
型注釈は`T[]`や`Array<T>`と書ける。要素には、異なる型を混ぜることができる。
```typescript
const arr2: (string | number | boolean)[] = ["foo", 123, true];
```

存在しない要素にアクセスしようとするとundefinedになるため、インデックスアクセスは極力使用しない。

#### readonly配列型
```typescript
const arr: readonly number[] = [1, 2, 3];
```
配列を引数として受け取る関数では、必要に迫られない限り**読み取り専用配列**とする。
pushやunshiftのような破壊的なメソッドは、読み取り配列に対しては使えない。

#### for-of文によるループ
for-of文が導入される以前は、同様の効果を持つforEachメソッドが利用されていた。
```typescript
const arr = [1, 2, 3];

for (const elem of arr) {
  console.log(elem); // 1, 2, 3
}

for (let elem of arr) {
  elem += 1;
  console.log(elem); // 2, 3, 4
}
```
各ループで変数が作り直されるため、constで問題はない。elemを書き換えても配列arr本体には影響しない。

#### タプル
タプル型は配列型の一種。あらかじめ要素の個数が判明している場合に適している。

### 分割代入
プロパティ名と同じ名前の変数への代入でよく用いられる。変数の型は型推論によって決められる。
```typescript
const { foo, bar } = obj;
```

##### 配列の分割代入
空白で要素をスキップできる。
```typescript
const [, foo, , bar, , baz] = arr;
```

##### 分割代入のデフォルト値
```typescript
const { foo = 123 } = obj;
```
undefinedの場合に適用される。nullの場合は、nullで代入される。

##### restパターン
残りのプロパティをすべて持つ。
```typescript
const { foo, ...rest } = obj;
```
イミュータブルな方法でオブジェクトを操作する際に活躍する。
あるオブジェクトから特定のプロパティを除いたオブジェクトを、新しいオブジェクトとして作成できる。

### その他の組み込みオブジェクト

#### Dateオブジェクト
操作しづらかったりミュータブルだったりで、使いにくいオブジェクトとして知られている。Temporalに置き換えが進んでいる。
UNIX時間は、1970年1月1日0時0分0秒からの経過時間をミリ秒で表したもの。

#### 正規表現

##### 正規表現フラグ

| フラグ | 意味                       |
|:----|:-------------------------|
| i   | 大文字小文字を区別しない             |
| g   | すべての箇所にマッチ               |
| m   | 先頭・末尾にマッチ                |
| s   | .に改行文字列を含む               |
| u   | 文字列をUnicodeコードポイント列として扱う |
| y   | 指定された開始位置からのみマッチ         |

##### replaceメソッド
マッチした箇所を置換し、新しい文字列を返す。

##### matchメソッド
マッチした部分文字列と、キャプチャリンググループにマッチした文字列の情報を返す。
キャプチャリンググループとは、正規表現中で使える `()` という構文のこと。

#### Mapオブジェクト
特定の値に対して対応する値を保持する。キーで保存されている値がない場合、返り値はundefinedになる。どんな型でもMapのキーに利用できる。

##### Mapとガベージコレクト
Mapのキーとして使われているオブジェクトは、Mapが存在する限りガベージコレクトされず、メモリ上に保持され続ける。
そういったキーを用いる場合には、WeakMapを利用する。

#### Setオブジェクト
集合を表すオブジェクト。

#### プリミティブにプロパティ？
```typescript
const str = "foo";
console.log(str.length); // 3
```
実はプリミティブに対してプロパティアクセスを行うたびに一時的にオブジェクトが作られている。
```typescript
type HasLength = {
  length: number;
};
const obj: HasLength = "foo";
```
HasLength型は、「number型のlengthプロパティを持つ値の型」であり、オブジェクトに制限されない。


## TypeScriptの関数

### 関数の作り方
ネストが深くならないように、早期リターンを活用する。

##### 返り値がない関数
return文は書かなくてもよいし、明記するなら `return;` と書く。

##### 関数式
関数は値（関数オブジェクト）であるため、変数に代入することができる。

##### 可変長式数
任意の数の引数を受け取れるようになる。型注釈が必要で、必ず配列型でなければならない。
```typescript
function sum(...args: number[]): number {
  return args.reduce((acc, cur) => acc + cur, 0);
}
```

##### コールバック関数
引数として関数を渡す。
```typescript
const names = users.map((u: User): string => u.name);
```
関数を変数に入れず、直接関数式を渡したほうがプログラムの見通しが良くなることが多い。
コールバック関数を引数として受け取るような関数は高階関数と呼ばれる。

##### 引数の型の省略
コールバック関数は多くの場合、引数の型を書かなくてもよい。

##### 関数型の記法
`(引数リスト) => 返り値の型`  
関数型中の引数名は、エディタの支援機能を充実させるために存在する。

##### 返り値の型注釈は省略すべきか
返り値に対して型チェックを働かせられる。明記したほうが有利なので返り値の型は必ず書く。

### 関数型の部分型
TSでは、型に合わせてプロパティ等が削られることはない。

#### 引数と返り値の型による部分型関係

![structural-subtype](/assets/2024-04-14-Introduction-to-TypeScript-for-future-professionals/structural-subtype.png)

```typescript
type HasName = {
  name: string;
};
type HasNameAndAge = {
  name: string;
  age: number;
};

// 返り値の型による部分型関係
const fromAge = (age: number): HasNameAndAge => {
  return { name: "Taro", age };
};
const f: (age: number) => HasName = fromAge;

// 引数の型による部分型関係
const showName = (obj: HasName): void => {
  console.log(obj.name);
};
const g: (obj: HasNameAndAge) => void = showName;
```

##### 引数の配列
`T[]`型を受け取る関数に、`readonly T[]`型を渡すことはできない。
内部で配列を破壊しないなら、引数の型は`readonly T[]`型にしたほうがよい。
関数の処理に必要最低限の型にすべき。

### ジェネリクス
型引数を持つ関数のこと。どんな型にも適用するロジックを持つ関数に適している。
```typescript
function identity<T>(arg: T): T {
  return arg;
}
```
型引数を持つ関数を呼び出すときは、型引数の指定を省略することができる。このときTは引数の方から推論される。

### 変数スコープ
- 内部のスコープからは、外部のスコープの変数にアクセスできる
- 同じ名前の変数がある場合、内側のスコープが優先される
- 変数のスコープは狭ければ狭いほど、見通しの良いプログラムになる
- `var`はブロックスコープに属さず、複数回宣言できる


## TypeScriptのクラス

### クラスの宣言と使用
クラスで作成されたオブジェクトは**インスタンス**と呼ばれる。また、クラスで宣言されているプロパティは**フィールド**とも呼ばれる。

#### クラスも変数である
関数やクラスもすべて「変数に入ったオブジェクト」として統一的に扱われる。
関数は関数オブジェクトと呼ばれ、クラスは**クラスオブジェクト**と呼ばれる。

#### コンストラクタ
`new User` 処理が完了した時点で、コンストラクタの処理も完了している。
コンストラクタ内でも`this`プロパティは使用できるが、初期化前は使用できない。
コンストラクタ内では、自身の読み取り専用プロパティにも代入できる。

#### アクセシビリティ修飾子
`private` は積極的に使う。
`let` と `const` の場合と同様に、機能が制限されているほうが、考慮すべきことが少なくなる分だけ優れいている。
`protexted` では、そのクラスを継承したクラスからもアクセスできる。

##### プライベートプロパティ
`#` はJSの機能なのでランタイムでもプライベート性が守られており、外から参照することができない。

### クラスの型
Userクラスを宣言すると、同じ名前の**User型**も作成される。

##### 変数名と型名の名前空間
変数名と型名は、別々の名前空間に属している。クラス宣言では、両方の名前空間に同時に同一名で作成される。

#### instanceof演算子と型の絞り込み
`john instanceof User` は「johnがUser型である」という判定ではなく、あくまで「johnがUserクラスのインスタンスである」という判定。
instanceofはランタイムにオブジェクトがどう作られたかを見ている。クラスという機構を前提としたinstanceofに頼るべきではない。
```typescript
function getPrice(customer: HasAge) {
  if (customer instanceof User) {
    if (customer.name === "foo") {
      return 0;
    }
  }
  return customer.age < 18 ? 1000 : 1800;
}
```
instanceofの型絞り込みの効果により、 if分の中ではcustomerはUser型になり、外ではHasAge型になる。
```typescript
if (customer instanceof User && customer.name === "foo") {
  return 0;
}
```
&&の左で行われた型の絞り込みは、後続の判定に適用される。

### クラスの継承
あるクラスの部分型となる別のクラスをつくれる。
継承はプログラムの設計を**複雑**にする。継承の効果的な使い方には確固たる答えがない。

#### 親機能の上書き
子クラスは、親の機能を上書きできる。

##### super
superは、親クラスのコンストラクタを呼び出すための特別な構文。子クラスでは呼び出しが必須。
コンストラクタの挙動を完全に書き換えることはできず、必ず親のコンストラクタを拡張する。
ただし、super呼び出しに与える引数をどのように用意するかは小クラスの自由。
```typescript
class PremiumUser extends User {
    rank: number;

    constructor(name: string, age: number, rank: number) {
        super(name, age);
        this.rank = rank;
    }
}
```

#### override修飾子
プロパティやメソッドに付加することで、それがオーバーライドであると宣言する。
overrideは必須ではなく、あくまでオーバーライドであることを明示するだけ。
「オーバーライドしたつもりができていなかった」等の事象を防げるため、極力利用する。
```typescript
class PremiumUser extends User {
  rank: number = 1;
  
  public override isAdult(): boolean {
    return true;
  }
}
```

#### privateとprotected
protectedを使用は、「子クラスによる自由な干渉を受け入れる」という意思表示を意味する。
極力privateや#を使用することで、バグが起きにくいプログラムにする。

#### implements
「クラスのインスタンスは与えられた型の部分型である」という宣言であり、型チェックの機能を担う。

### this
クラスメソッドの関数オブジェクトは、**複数のインスタンス間で共有**される。
インスタンスをいくつ作っても同じ中身の関数オブジェクトが量産されないため、**経済的**。
それでもなお**インスタンスごとに異なる挙動**を実現できるのは、thisが存在するおかげ。

### 例外処理
エラーの発生箇所（スタックトレース含む）を記録するのがErrorオブジェクトの役割。

##### 例外処理と大域脱出
処理の失敗は、「例外を使う」方法と「失敗を表す返り値を使う」方法でハンドリングされる。どちらも一長一短。

#### try-catch文
ネストしたtry-catch分で囲まれていた場合、一番内側で例外はキャッチされる。
多重のtry-catch文は、プログラムの理解が難しくなるため望ましくない。

##### finally
finallyブロックは、return文による脱出に割り込むことができる。

#### throw
throw文はどんな値でも投げられる。基本的にはErrorオブジェクトを投げた方がよい。


## 高度な型

### ユニオン型とインターセクション型
静的型付け言語は多くあるが、これらの機能を持つ言語はあまりない。

##### ユニオン型
型Tまたは型U。

##### インターセクション型
型Tかつ型U。`&`で作られた型、それぞれの構成要素の型の部分型になる。

#### ユニオン型とインターセクション型の表裏一体な関係
条件演算子の結果は真の場合と偽の場合の型のユニオン型になる。
関数型同士のユニオン型を合成して新たな関数型を作る際、返り値の型は共変の位置にあるためユニオン型となり、引数の型は反変の位置にあるためインターセクション型になる。
```typescript
type getName = ({ name: string }) => string;
type getSpecies = ({ species: string }) => number;

const mysteryFunc = Math.random() < 0.5 ? getName : getSpecies;
// ({ name: string }) => string) | ({ species: string }) => number)
// (arg0: { name: string; } & { species: string; }) => string | number

const value = mysteryFunc({ name: "foo", species: "bar" });
```

#### オプショナルプロパティ再訪
`age?: number` と `age: number | undefined` は意味が異なる。前者は「ageの不在」が許容され、後者は許容されない。
後者の場合、undefinedでもいいので明示的にageが存在する必要がある。
```typescript
type Human = {
  name: string;
  age: number | undefined;
};

// 以下はageがなくてエラー
const taro: Human = {
  name: "Taro",
};
```
**省略に強い動機**がないのであれば、`age?: number` よりも `age: number | undefined` の方が適している。

`exactOptionalPropertyTypes` を有効化すると、`age?: number` の `?` は「ageの不在」のみを表すようになる。
1つの記法は1つだけ意味を持ったほうが、プログラムがより明確になるため有効化が推奨される。

#### オプショナルチェイニング
`obj?.prop` では、objがnullやundefinedの場合でもランタイムエラーは発生せず、結果は `undefined` になる。
`?.` はそれ以降のプロパティアクセス・メソッド呼び出しをまとめて飛ばす効果を持つ。

### リテラル型

#### 4種類のリテラル型
リテラル型は、**プリミティブ型をさらに細分化した型**。
`"foo"` という型も一種のリテラル型。`"foo"` という型の意味は「`"foo"`という文字列のみが属する型」。
```typescript
const foo: "foo" = "foo"; // 文字列のリテラル型
const zero: 0 = 0;        // 数値のリテラル型
const truth: true = true; // 真偽値のリテラル型
const three: 3n = 3n;     // bigintのリテラル型
```

#### ユニオン型とリテラル型を組み合わせ
リテラル型のユニオン型は、TSの頻出パターンのひとつ。
```typescript
function signNumber(type: "plus" | "minus") {
    return type === "plus" ? 1 : -1;
}
```

#### リテラル型のwidening
`let` では、プリミティブ型に変換される。`let` と同様にあとから書き換え可能なため、オブジェクトリテラルでもwideningが発生する。
```typescript
const uhyo1 = "uhyo"; // "uhyo"型
let uhyo2 = "uhyo"; // string型

const uhyo = {
    name: "uhyo", // string型
};
```
コンパイルエラー等の中でもwideningが起こる。ユーザにわかりやすいようにwideningされている。

### 型の絞り込み
型を特定するコードを書くことで、ユニオン型の型情報が変化する。

#### typeof演算子の絞り込み
typeof演算子にプリミティブ値を与えると、その種類に応じた文字列が返される。

| 式の評価結果    | typeof式の結果  |
|:----------|:------------|
| 文字列       | `"string"`    |
| 数値        | `"number"`    |
| 真偽値       | `"boolean"`   |
| BigInt    | `"bigint"`    |
| シンボル      | `"symbol"`    |
| null      | `"object"`    |
| undefined | `"undefined"` |
| オブジェクト    | `"object"`    |
| 関数        | `"function"`  |

#### 代数的データ型をユニオン型で再現する
代数的データ型は非常に強力な機構で、「扱うデータの型と可能性を型で正確に表現する」ことに貢献する。
「**判別用の情報（タグ）**」を持つのが代数的データ型の特徴であり、TSではタグを「リテラル型を持つプロパティ」として表現する。
複数種類のデータが混ざる場合、それぞれを**別々に型で定義**し、それぞれに**タグを付与**する。
TSにおける極めて基本的な設計パターン。
```typescript
type Animal = {
  tag: "animal";
  species: string;
};
type Human = {
  tag: "human";
  name: string;
};
type User = Animal | Human;

const tama: User = {
  tag: "animal",
  species: "cat",
}
const uhyo: User = {
  tag: "human",
  name: "uhyo",
}

function getUserName(user: User) {
  if (user.tag === "human") {
    return user.name;
  }
  return "名無し";
}

console.log(getUserName(tama)); // 名無し
console.log(getUserName(uhyo)); // uhyo
```

#### switch文の絞り込み
Userの定義変更と同時に変更すべき箇所が明らかになるため、switch文を使うほうが有利。
```typescript
function getUserName(user: User) {
  switch (user.tag) {
    case "human":
      return user.name;
    case "animal":
      return "名無し";
  }
}
```
`noImplicitReturns` を有効化すると、「return漏れ」がコンパイルエラーになる。

### lookup型・keyof型

#### lookup型
`T[K]` は、オブジェクト型Tのプロパティ名Kに対応する型。「同じことを二度書かない（DRY）」（**型情報の再利用**）のためにlookup型を使う。
```typescript
type Human = {
  type: "human";
  age: number;
};

function setAge(human: Human, age: Human["age"]) {
  return {...human, age};
}
```

#### keyof型
`keyof T` は、オブジェクト型Tのプロパティ名のリテラル型のユニオン型。
```typescript
type Human = {
  name: string;
  age: number;
};
type HumanKeys = keyof Human; // "name" | "age"

const key: HumanKeys = "name";
```

##### typeofとの組み合わせ
`keyof` と `typeof` の組み合わせで、オブジェクトのプロパティ名のリテラル型のユニオン型を取得できる。実装を変えても型定義が自動的に変わる。
```typescript
const mmConversionTable = {
  mm: 1,
  m: 1000,
  km: 1000000,
};

type types = typeof mmConversionTable; // { mm: number; m: number; km: number; }
type keys = keyof typeof mmConversionTable; // "mm" | "m" | "km"
```

#### keyof型とlookup型のジェネリクス
keyofは型引数に対しても有効に働く。
```typescript
function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

### 型アサーション
型アサーションの使用はできるだけ避ける。`!` や `as` などを使わなくていいような書き方をするのもTS力のひとつ。

##### as
`式 as 型` という構文で、その式の型を強制的に変える。asはTSの判断を強制的に覆すための機能であり、誤った使い方をすると型安全性の破壊につながる。

##### 正しい使い方
**不正確な型を正しく直す**ために使う。TSがカバーできない型の絞り込みをプログラマーがasを使って代行する。asを使う場合はコメントにその理由を残しておく。
```typescript
function getNamesIfAllHuman(users: readonly User[]): string[] | undefined {
  if (users.every((user) => user.tag === "human")) {
    return (users as Humna[]).map((user) => user.name);
  }
  return undefined;
}
```

##### !
`式!` という構文で、式がnullまたはundefinedである可能性を無視する。asと同様に型安全性の崩壊につながる危険性を秘めている。

##### as const
各種リテラルを「変更できない」ものとして扱う。値をデータの源としたい場合に活躍する。
```typescript
const names1 = ["foo", "bar"];          // string[]型
const names2 = ["foo", "bar"] as const; // readonly ["foo", "bar"]型
type Name = (typeof names2)[number];    // "foo" | "bar"
```

### any型・unknown型
##### any型
最凶の危険性を誇る。「**型チェックを無効化**」する型。ランタイムエラーの危険性が大きく増加する。
JSからTSへの移行を支援するための型。

##### unknown型
何でも入れられる型。値が正体不明であるため、できることが限られる。

### さらに高度な型

#### object型・never型

##### object型
「プリミティブ以外すべて」を表す型。オブジェクトのみに制限したい場合に使用する。

##### never型
「当てはまる値が存在しない」を表す型。never型の値が存在しているコードは実際には実行されない。never型はすべての型の部分型である。
```typescript
function thrower(): never {
  throw new Error();
}
const result: never = thrower(); // コンパイルエラーにならない
```
関数throwerの返り値を得ることは不可能なため、返り値の型はnever型になる。

#### 型述語（ユーザー定義型ガード）
`引数名 is 型` という構文。ユーザー定義型ガードの返り値は真偽値で、trueなら指定した型になる。関数を用いて型を絞り込みを行う唯一の手段。
```typescript
function isStringOrNumber(value: unknown): value is string | number {
  return typeof value === "string" || typeof value === "number";
}
```

#### 組み込みの型

##### `Readonly<T>`
すべてのプロパティを読み取り専用にする。
```typescript
type Human = {
  name: string;
  age: number;
};
type ReadonlyHuman = Readonly<Human>; // { readonly name: string; readonly age: number; }
```

##### `Partial<T>`
すべてのプロパティをオプショナルにする。
```typescript
type PartialHuman = Partial<Human>; // { name?: string | undefined; age?: number | undefined; }
```

##### `Pick<T, K>`
Kというプロパティ名のみを持つ型を作る。
```typescript
type PickHuman = Pick<Human, "name">; // { name: string; }
```

##### `Omit<T, K>`
Kというプロパティ名を除いた型を作る。
```typescript
type OmitHuman = Omit<Human, "age">; // { name: string; }
```


## TypeScriptのモジュールシステム
１ファイル１モジュール。

### import宣言とexport宣言
インポートされたモジュールは実行される。インポートされた側のモジュールが先に実行される。

#### 変数のエクスポート・インポート
import宣言はどこに書いてもよい。`内側の変数名 as 外側の変数名` を用いて変数名を変えながらエクスポートすることができる。
```typescript
const name = "foo";
export { name as myName };
```

##### モジュールとカプセル化
モジュールはカプセル化の手段をとして使える。
モジュール内で定義された変数はそのモジュール内をスコープとする。
モジュールの内部実装を外部から**隠蔽**できる。

##### モジュールの実体は１つ
モジュールは、複数回・複数の場所でインポートされたとしても**実体は１つ**。

#### defaultエクスポート・インポート
`export default 式;` とは、暗黙のうちにdefaultという名前の変数を用意してエクスポートする機能。
モジュールからエクスポートされる代表的な値を表すためにdefaultという名前が使用される。

defaultエクスポート・インポートは、入力補完等の機能が十分に働かないため使うべきではない。
エクスポートする変数名を決めるときはそのままインポートできる名前にすべき。

#### 型のエクスポート・インポート
`export type {}` と `import type {}` では、型としてのみ使用可能になる。
この構文を用いると、JSへの変換時に削除しても良いことが明確になる。

#### その他

##### 一括インポート
`import * as 変数名 from "モジュール名";` では、変数名にモジュール名前空間オブジェクト（そのモジュールからエクスポートされた変数すべてをプロパティに持つ特殊なオブジェクト）が代入される。

##### 再エクスポート構文
`export { 変数名リスト } from "モジュール名";`

##### スクリプトとモジュール
TS・JSファイルは、スクリプトとモジュールの2種類に分類される。import・exportは、モジュールのみで使用できる。
スクリプトではトップレベルに定義された変数や型のスコープがファイルの中だけではなくプロジェクト全体に広がる。
スクリプトではなくモジュール扱いしたい場合は、`export {};` と書く。

### @typesとDefinitelyTyped
型定義が同梱されていないパッケージは、コンパイルエラーになる。
@typesパッケージは同梱されていない型定義を補う。
@typesパッケージはコンパイル時にのみ必要なため、devDependenciesに入れる。
Microsoftが運営するDefinitelyTypedが、@typesパッケージの開発・運用を行っている。


## 非同期処理

#### 非同期処理とは
CPUやメモリよりも外に出る処理は時間がかかる。

##### シングルスレッドモデル・ノンブロッキング
ブロッキングな処理とは、その処理の実行が完了するまでプログラムがそこで停止するような処理のこと。
JS・TSは**シングルスレッド**なモデル（プログラムの複数箇所を同時に（並列に）実行できない）のため、時間がかかる処理にブロッキングされるのは歓迎されない。

#### 非同期処理とコールバック関数

##### コールバック関数とは
非同期処理が終わったときに呼び出される関数のこと。
非同期処理の裏で、プログラムが次に進むことができる。
プログラムが最後まで到達しても、非同期処理が残っているとプログラムは終了しない。

##### 同期処理と非同期処理の順序
同期的に実行中のプログラムに非同期処理が割り込むことはない。
同期的な実行がすべて終了してから、コールバック関数が呼び出される。

#### Promise
Promiseは、**非同期処理そのもの**を表す抽象的なオブジェクト。
Promiseオブジェクトが返された時点で、すでに非同期処理は開始している。
Promiseの結果が決まることを「Promiseの解決」と呼ぶ。

##### Promiseのエラー処理
catchのコールバックの引数には、常にunknown型という型注釈をつける。

##### Promiseオブジェクトの自作
`resolve` を呼び出すとPromiseが成功し、`reject` を呼び出すとPromiseが失敗する。

#### Promiseの静的メソッド

##### Promise.all
すべて成功したら成功、どれか１つでも失敗したら失敗を返す。

##### Promise.allSettled
すべての終了を待ち、すべてのPromiseの結果を返す。
```typescript
[
  { status: "fulfilled", value: 1 },
  { status: "rejected", reason: new Error() },
]
```

##### Promise.race
成功・失敗を含め、一番早く出た結果を返す。

##### Promise.any
一番早く成功した結果を返す。

#### async/await構文
awaitはasync関数の中でのみ使用できる。
awaitはPromiseの結果を待つと同時に、async関数の実行を中断し、他の場所（同期的な実行の続きや別の非同期処理）に制御を移す。
「async関数がawaitを待っている時間」は、呼び出し元から見ると「async関数が返したPromiseを待っている時間」の一分になる。


## TypeScriptのコンパイラオプション

##### 新規プロジェクトでのお勧め設定
安全性という観点ではなるべく厳しい設定にするほうがよく、厳しい設定のほうが罠が少ないためむしろ初心者には望ましい。

| オプション名 | 説明                        |
|:----------|:--------------------------|
| strict | 重要なコンパイルオプショングループ         |
| noUncheckedIndexedAccess | インデックスシグネチャがオプショナルとして扱われる |
| exactOptionalPropertyTypes | ?とundefinedが完全に区別される      |
| noImplicitReturns | returnが必須になる              |
| noFallthroughCasesInSwitch | switch文でbreakが必須になる       |
| noImplicitOverride | オーバーライドの明記が必須になる          |
