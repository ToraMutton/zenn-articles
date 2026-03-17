---
title: "let/constは知ってるけど、typeって何？TypeScriptの型定義をCanvas作りながら整理した"
emoji: "💠"
type: "tech"
topics: ["javascript", "typescript"]
published: true
---

## はじめに

こんにちは、トラマトです！
2週間前くらいからJavaScriptを勉強し始めて、それなりに書けるようになってきました。

しかしTypeScriptを触り始めたとき、こんなコードで一瞬止まってしまいました。
```typescript
type Params = {
  mode: string;
  points: number;
  waves: number;
};
```

**`type`って何？`let`や`const`と何が違うの？**

今回はCanvas APIを使って動く幾何学アートを作りながら、この疑問を整理しました。

---

## まず前提：JSの変数宣言おさらい

TypeScriptの話をする前に、JavaScriptの変数宣言を軽く振り返ります！

```javascript
var name = "寅松";   // 古い書き方で、自分は使ったことがありません
let count = 0;           // 変更できる変数
const PI = 3.14;         // 変更できない定数
```
`var`は同名変数が作れてしまうのでバグの温床らしいんですよね。今後使うことはないでしょう。
`let`と`const`は値を入れる**箱**を作るもの。箱に何を入れるかは私たちが自由に決められるんですよね。。
JavaScriptはここがゆるくて、こんなことも普通にできてしまう…

```javascript
let score = 100;
score = "百点";  // 数値に文字列を入れても怒られない
```

大学で**C言語**を学んだ後なので、最初は脳が理解を拒みましたがすぐ慣れました。

---

## TypeScriptが追加するもの：型注釈

TypeScriptでは変数に**型注釈**を書けるんですよ。いやー革命。

```typescript
let score: number = 100;
score = "百点";  // ← コンパイルエラー！
```

こんな感じで`変数名: 型名`という書き方で「この箱には数値しか入れないぜ！」と宣言できます。
さらに、例えば以下のようにも書けますね。

```typescript
let name: string = "寅松";
let age: number = 19;
let isActive: boolean = true;
```
### 実は毎回書かなくてもOK：型推論
ちなみに、毎回`: number`って書くの面倒って思いませんか？
実はTypeScript君は最初の値を入れた時点で空気を読んでくれるんです。

```typescript
let score = 100; // 「あ、100が入ったからこの箱はnumber型だ！」と自動判定
score = "百点";  // ← もちろんコンパイルエラーになる！
```

これを型推論と呼びます。全ての変数に型を書かなくても勝手に推測して守ってくれるの結構便利じゃないですか？

まぁ、ここまではいいんですよ。問題は**オブジェクト**を扱うとき。

```typescript
// これだと毎回同じ形を書かないといけない
let user: { name: string; follower: number } = { name: "Twitter", follower: 602 };
let admin: { name: string; follower: number } = { name: "Github", follower: 14 };
```

同じ形のオブジェクトを使う事に毎回`{ name: string; follower: number }`と書くのは面倒だし、ミスのもとになる！
…のですが、これを解決するのが`type`と`interface`です。

---

## `type` で形に名前をつける

`type`は**オブジェクトの形(構造)に名前をつける**ための構文です。

```typescript
type Sns = {
  name: string;
  follower: number;
};

let tw: Sns = { name: "Twitter", follower: 602 };
let gh: Sns = { name: "Github", follower: 14 };
```

`Sns`という名前で形を定義しておけば、あとは`Sns`と書くだけでいいという大革命。
**設計図に名前をつけたイメージ** ですね。

`type`はオブジェクトだけじゃなく、こんな使い方もできます。便利〜
```typescript
// 文字列のユニオン型(どれかひとつ)
type Axis = "x" | "y" | "z";

// 型に別名をつける
type ID = number;
```

---

## `interface` も似たようなもの

`interface`も**オブジェクトの形を定義する**ための構文なのですが、見た目がちょっと違います。
```typescript
interface Sns {
  name: string;
  follower: number;
};
```

これだけ見ると`type User = { ... }`と`interface User { ... }`で、`=`に注意が必要ですが、ほぼ同じ書き方で同じことができますね。

じゃあ何が違うのかといいますと、**interfaceは後から拡張できます**。
```typescript
interface Sns {
  name: string;
}

interface Sns {
  follower: number;  // 同じ名前で追加できる！(宣言マージ)
}

// Snsは { name: string; follower: number } になる
```

ちなみに`type`で同じことをやるとエラーになります。
```typescript
type Sns = { name: string; };
type Sns = { follower: number; };  // エラー！重複定義はダメ
```

自分1人で開発するレベルなら基本`type`使っておけば問題なさそうかなぁと勝手に思ってます。

---

## TypeScriptの正体

ちなみに、ここが一番面白くて混乱したところなんですが、この`type`や`interface`、実行時には**跡形もなく消滅します**。

C言語の構造体だとメモリのレイアウトなど実体としての役割もありますが、なんとTypeScriptの型定義は、**コードを書いてる時にエラーチェックするためだけの概念**なんです。
TypeScriptをビルドしてJavaScriptに変換(コンパイル)すると、型定義の部分はバッサリ削られます。

「じゃあ誰がエラーチェックしてるん？」って話ですが、裏では **TypeScriptコンパイラ (tsserver)** というものが動いています。
人間がコードを書いている最中に、このtsserverが「この変数にはこの型しか入らないぞ！」と見張っています。
そしてミスした場合に、LSP (Language Server Protocol) という仕組み（エディタとtsserverを繋ぐ伝書鳩みたいなもの）を通して、VSCodeやNeovimなどのエディタに「そこ、型が違うぞ！」とリアルタイムで警告を出してくれるわけです。いやぁ最高ですね。

---

## 実際のコードで見てみる

今回作ったCanvas描画コンポーネントでは、こう使ってます。
```typescript
type Params = {
  mode: string;
  points: number;
  waves: number;
  waveHeight: number;
  baseRadius: number;
  rotationSpeed: number;
  waveSpeed: number;
  fadeOpacity: number;
};

const [params, setParams] = useState<Params>({
  mode: 'Chaos',
  points: 890,
  // ...
});
```

`useState<Params>`と書くことで、「このstateはParams型のオブジェクトを持つぞ！」とTypeScriptに教えられます。
これによって、`params.points`に文字列を入れようとしたり存在しないキーにアクセスしようとしたりすると、コードを書いた時点でエディタがエラーを出してくれます。

さらに最大のメリットがあって、それは**圧倒的にサボれる**こと。
`params.`と打った瞬間に、「次は`waveHeight`とか`rotationSpeed`を使うんだろ？」とエディタが候補を全部出してくれるんです。
自分が苦手な短期記憶力に頼らなくていいし、タイピングミスで画面が真っ白になる悲しい時間がゼロになります。
これこそがTypeScriptの真のパワーですね。

---

## まとめ

| 構文 | ターゲット | いつ決まる？ | 役割 |
|------|------|--------|--------|
| `var` | なし | - | 古代の遺物 |
| `let` / `const` | **値** (データそのもの) | 実行中 (JS) | 箱を作ってデータを入れる |
| `type` / `interface` | **型** (データの形) | 書いてる時 (TS) | 箱のルールを決める(設計図) |

`let`と`const`が**実行時に使う箱**を作るのに対して、`type`と`interface`は**書く時にエディタがチェックするための設計図**を作るもので、かなり違う概念でした。

JavaScriptと違って、TypeScriptを使うと「実行してみたら動かなかった😭」ということが減る気がします。

今回のプロジェクトはまだまだ未完成で、さっきの`Params`型も大改造する予定です。(進捗は[Github](https://github.com/ToraMutton/geo-art-gen)にて)

それでは〜
