---
layout:        post
title:         "Deep TypeScript"
date:          2025-10-04 00:00:00 +0900
modified_date: 2025-10-14 00:00:00 +0900
categories:    [tech, lang]
tags:          [cheating]
---

![thumbnail](/assets/2025-10-04-deep-typescript/thumbnail.png)

## prototype
JavaScriptは**プロトタイプベース**の言語。別のオブジェクト（＝プロトタイプ）を参照して性質を引き継ぐ。
あるオブジェクトにプロパティやメソッドが見つからなければ、プロトタイプをたどって探す。
prototypeに定義された関数はインスタンス間で共有され、メモリ効率も良い。

### サンプル
[class構文](####class構文)の実態は[prototype構文](####prototype構文)である。

#### class構文

```typescript
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(`${this.name} makes a sound`);
  }
}
```

```typescript
class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }
  
  bark() {
    console.log(`${this.name} barks!`);
  }
}
```

#### prototype構文

```typescript
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function() {
  console.log(this.name + " makes a sound");
};

// Animal 概念
/*
  [[Prototype]]: Function.prototype,
  [[Call]]: (thisArg, [name]) => {
    // Animal("Pochi")
    thisArg.name = name;
    return undefined;
  },
  [[Construct]]: ([name], newTarget) => {
    // new Animal("Pochi")
    const obj = OrdinaryCreateFromConstructor(newTarget, Animal.prototype);
    Animal.[[Call]](obj, [name]);
    return obj;
  },
  prototype: {
    [[Prototype]]: Object.prototype,
    constructor: Animal,
    speak: function() {
      console.log(this.name + " makes a sound");
    }
  },
  name: "Animal",
  length: 1 // 仮引数の数（name）
*/
```

```typescript
function Dog(name, breed) {
  Animal.call(this, name);
  this.breed = breed;
}
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
Dog.prototype.bark = function() {
  console.log(this.name + " barks!");
};

// Dog 概念
/*
  [[Prototype]]: Function.prototype,
  [[Call]]: (thisArg, [name, breed]) => {
    // Dog("Pochi", "Shiba")
    Animal.[[Call]](thisArg, [name]);
    thisArg.breed = breed;
    return undefined;
  },
  [[Construct]]: ([name, breed], newTarget) => {
    // new Dog("Pochi", "Shiba")
    obj = OrdinaryCreateFromConstructor(newTarget, "%Object.prototype%", internalProto = Dog.prototype);
    Dog.[[Call]](obj, [name, breed]);
    return obj;
  },
  prototype: {
    [[Prototype]]: Animal.prototype,
    constructor: Dog,
    bark: function() {
      console.log(this.name + " barks!");
    }
  },
  name: "Dog",
  length: 2 // 仮引数の数（name, breed）
*/
```

```typescript
const dog = new Dog("Pochi", "Shiba");
dog.speak(); // Pochi makes a sound
dog.bark();  // Pochi barks!

// dog 概念
/*
  [[Prototype]]: Dog.prototype,
  name: "Pochi",
  breed: "Shiba"
*/

// dog 全体像
/*
{
  [[Prototype]]: {
    [[Prototype]]: {
      [[Prototype]]: Object.prototype,
      constructor: Animal,
      speak: [Function: speak]
    },
    constructor: Dog,
    bark: [Function: bark]
  },
  name: "Pochi",
  breed: "Shiba"
};
*/
```

### Object.getPrototypeOf
`prototype`を取得する。

#### インスタンスオブジェクト

```typescript
Object.getPrototypeOf(dog) === Dog.prototype;                   // true
Object.getPrototypeOf(Dog.prototype) === Animal.prototype;      // true
Object.getPrototypeOf(Animal.prototype) === Object.prototype;   // true
Object.getPrototypeOf(Object.prototype) === null;               // true
```

#### コンストラクタオブジェクト

```typescript
Object.getPrototypeOf(Dog) === Function.prototype;              // true
Object.getPrototypeOf(Function.prototype) === Object.prototype; // true
Object.getPrototypeOf(Object.prototype) === null;               // true
```

### instanceof
インスタンスオブジェクトが特定のコンストラクタオブジェクトから生成されたかどうかを判定する演算子。

#### サンプル

```typescript
dog instanceof Dog;       // true
dog instanceof Animal;    // true
dog instanceof Object;    // true
```

#### 内部処理イメージ
プロトタイプチェーンをたどって判定。

```typescript
function instanceOf(left, right) {
  const proto = right?.prototype;
  let cur = Object.getPrototypeOf(left);
  while (cur) {
    if (cur === proto) return true;
    cur = Object.getPrototypeOf(cur);
  }
  return false;
}
```

## this

### 種類

##### デフォルトバインディング
this → strict: `undefined` / 非strict: `window`（または `global`）

```typescript
function showMessage() {
  console.log(this?.message || "no this");
}
const message = "global";
showMessage(); // strict: no this / 非strict: global
```

##### 暗黙的バインディング
this → 呼び出し元のオブジェクト

```typescript
const user = {
  name: "Alice",
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  }
};
user.greet(); // "Hello, I'm Alice"
```

##### 明示的バインディング
this → call / apply / bind の第1引数

```typescript
function introduce() {
  console.log(`I'm ${this.name}, ${this.age} years old.`);
}
const person = { name: "Bob", age: 25 };
introduce.call(person); // "I'm Bob, 25 years old."
```

##### コンストラクタ呼び出し（暗黙的バインディングの一種）
this → 新しく生成されたオブジェクト  
※以下の例では `increment` がインスタンスごとに生成されるため、メモリ効率が悪い。通常は `Counter.prototype` に定義する

```typescript
function Counter() {
  this.count = 0;
  this.increment = function() {
    this.count++;
    console.log(this.count);
  };
}
const c = new Counter();
c.increment(); // 1

// c 概念
/*
  [[Prototype]]: Counter.prototype,
  count: 0,
  increment: function() {
    this.count++;
    console.log(this.count);
  }
*/

// c 全体像
/*
  [[Prototype]]: {
    [[Prototype]]: Object.prototype,
    constructor: Counter,
  },
  count: 0,
  increment: function() {
    this.count++;
    console.log(this.count);
  }
*/
```

##### クラスのインスタンスメソッド（暗黙的バインディングの一種）
this → そのクラスのインスタンス  
※詳しい仕組みは [prototype](##prototype) や [Array](###Array) 参照のこと

```typescript
class Player {
  constructor(name: string) {
    this.name = name;
    this.hp = 100;
  }
  attack() {
    this.hp -= 10;
    console.log(`${this.name} attacked! HP: ${this.hp}`);
  }
}
const p = new Player("Rin");
p.attack(); // "Rin attacked! HP: 90"
```

##### アロー関数（レキシカル this）
this → 定義されたときの外側スコープの this

```typescript
const team = {
  name: "Red",
  members: ["Ken", "Mia"],
  showMembers() {
    this.members.forEach(member => {
      console.log(`${member} from ${this.name}`);
    });
  }
};
team.showMembers(); 
// "Ken from Red", "Mia from Red"
```

##### DOM イベントハンドラ
this → イベントを受け取った要素

```typescript
document.querySelector("button").addEventListener("click", function() {
    this.textContent = "Clicked!";
    console.log(this); // <button>Clicked!</button>
});
```

### Array
JavaScriptのArrayはオブジェクトであり、ブラケット記法で要素にアクセスする。
`push`や`pop`はArrayオブジェクトのメソッドであり、**暗黙的バインディング**により`this`は配列オブジェクトを指す。

```typescript
const arr = [10, 20, 30];
arr.push(40);
console.log(arr[3]); // 40

// Array 概念
/*
  [[Prototype]]: Array.prototype,
  0: 10,
  1: 20,
  2: 30,
  length: 3,
*/

// Array 全体像
/*
  [[Prototype]]: {
    push: function(...items) {
      let len = this.length >>> 0;
      for (let i = 0; i < items.length; i++) {
        this[len] = items[i];
        len++;
      }
      this.length = len;
      return len;
    },
    pop: function() {
      let len = this.length >>> 0;
      len--;
      const value = this[len];
      delete this[len];
      this.length = len;
      return value;
    }
  },
  0: 10,
  1: 20,
  2: 30,
  length: 3,
*/
```


## closure
外部の変数情報を保持した関数のこと。
外部の変数情報は**レキシカル環境**に保持される。

```typescript
function createAdder(a: number) {
  return function add(b: number) {
    const sum = a + b;
    return sum;
  }
}

// createAdder 概念
/*
  [[Prototype]]: Function.prototype,
  [[Call]]: (thisArg, [a]) => {
    // createAdder(2)
    const add = function add(b) {
      const sum = a + b;
      return sum;
    };
    return add;
  },
  prototype: { constructor: createAdder },
  name: "createAdder",
  length: 1
*/
```

```typescript
const addTo2 = createAdder(2);
addTo2(5); // 7

// addTo2 概念
/*
  [[Prototype]]: Function.prototype,
  [[Call]]: (thisArg, [b]) => {
    // addTo2(5)
    const sum = [[Environment]].a + b; // 2 + 5
    return sum;
  },
  [[Environment]]: {
    // 実行時のレキシカル環境
    a: 2,             // 環境レコード
    outer: GlobalEnv  // 外側のレキシカル環境への参照
  },
  name: "add",
  length: 1
*/
```


## asynchronous

### Promise
「未来の結果を入れる箱」を作る。
`resolve()` が呼ばれると「成功」状態に、 `reject()` が呼ばれると「失敗」状態になる。

```typescript
const promise = new Promise((resolve, reject) => {
  if (Math.random() < 0.5) resolve("成功しました");
  else reject("失敗しました");
});

promise
  .then(result => console.log(result))
  .catch(error => console.error(error));
```

### setTimeout
「一定時間後に関数を実行する予約」を行う。
呼び出し時点で関数はまだ実行されず、タイマーを設定して即座に戻る（ノンブロッキング）。

```typescript
setTimeout(() => {
  console.log("1秒後に実行されました");
}, 1000);
```

### clearTimeout
`setTimeout` の返す「タイマーID」を `clearTimeout` に渡して呼ぶと、その予約はキャンセルされる。

```typescript
const timerId = setTimeout(() => {
  console.log("これは実行されません");
}, 3000);

clearTimeout(timerId);
```

### AbortController

##### controller.signal
スイッチの状態を監視するオブジェクト。

##### controller.abort()
中断スイッチ。 `signal.aborted` が `true` になり、 `"abort"` イベントが発火する。

```typescript
const controller = new AbortController();
const { signal } = controller;

signal.addEventListener("abort", () => {
  console.log("中断されました！");
});

setTimeout(() => {
  controller.abort();
}, 2000);
```
