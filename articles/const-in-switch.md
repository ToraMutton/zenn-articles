---
title: "switch文でconstは使えない！警告の正体と解決法"
emoji: "‼️"
type: "tech"
topics: ["javascript", "typescript", "react"]
published: true
---

## はじめに

こんにちは、トラマトです！
現在、Canvas APIを使ってパラメータをいじれる幾何学アートジェネレーター（[Github](https://github.com/ToraMutton/geo-art-gen)）を作っています。

しかし、描画のアルゴリズムを`switch`文で切り替えようとしたとき、こんなエラーが出てきました。

`Unexpected lexical declaration in case block`

え、caseの中で変数宣言しちゃダメなんすか？と最初困惑したのですが、原因と解決法、JavaScriptの仕様に納得したので備忘録としてまとめます！

---

## 遭遇したエラー

幾何学模様の形を`mode`という変数で切り替えるために、こんなコードを書いていました。

このエラーの正体はTypeScriptのコンパイルエラーではなく、ESLint(コードの問題点を指摘してくれる静的解析ツール)の`no-case-declarations`というルールによる警告です。

```typescript
switch (mode) {
  case 'Polygon': // 多角形モード
    const sides = Math.max(3, waves);
    const a = Math.PI / sides;
    // ...描画処理...
    break;

  case 'Heart': // ハート型モード
    const r = baseRadius * 0.1;
    // ...描画処理...
    break;
}
```

---

## 原因: switch文は全部ぶち抜く仕様だった

ちょっと前に大学でC言語を触っていた感覚だと、「`case`ごとに処理は別なんだから、変数も別物なのでは？」と思ってしまいます。
しかしなんと、JavaScript(TypeScript)の仕様では、`switch`**文全体で1つのスコープを共有**していたのです。

```typescript
// ↓この { から
switch (mode) { 
  case 'Polygon':
    const sides = 3;
    const a = 10;
    break;

  case 'Heart':
    const r = 5; // ← 同じ空間に sides, a, r が混在してしまっている
    break;
// ↓この } までが同じ1つの空間
}
```

`const`や`let`は**ブロックスコープ(`{}`の中だけで有効な変数)** を作るための宣言です。
しかし、このままでは`Polygon`用の変数も`Heart`用の変数も同じ部屋に散らかっている状態になり、関数名の衝突などのバグの温床になります。
だからしっかりと**Unexpected lexical declaration**という警告を出してくれていたわけですね〜

---

## 解決法: caseごとにブロックを作る

原因が「同じ部屋だから」なので、解決法は至ってシンプル。以下のように`case`の中に**壁を作ってあげればOK**です。

```TypeScript
switch (mode) {
  case 'Polygon': {  // ← { を追加してブロックスコープを作る！
    const sides = Math.max(3, waves);
    const a = Math.PI / sides;
    break;
  }                  // ← } で閉じる

  case 'Heart': {    // ← ハート用にも個室を作る
    const r = baseRadius * 0.1;
    break;
  }                 // ← こっちも閉じる！
}
```

`case`の中身を`{ }`で囲むだけ！
これだけで独立したブロックスコープが生まれるので、例えば`Polygon`の部屋で作った`sides`という変数が他の部屋に影響を与えなくなります。エラーも綺麗に消え去りますね。

---

## まとめ

- `switch`文の各`case`は、デフォルトでは**同じスコープ**を共有している
- これにより、そのまま`const`や`let`を使うとスコープが混ざりエラーを吐く
- `case`の中身を`{ }`で囲うだけで独立したスコープとなり、安全な変数宣言が可能になる

基礎的なことですが、TypeScriptのエラーから「なぜダメなの？？」を深堀りすると、言語についてより詳しくなれるような気がするので面白いですね。

引き続き、幾何学アートジェネレーターの完成を目指してゴリゴリ書きまくっていきます！

それでは！
