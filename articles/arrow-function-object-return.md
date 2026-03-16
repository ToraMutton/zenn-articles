---
title: "アロー関数でオブジェクトを返すときの ({}) って何者？"
emoji: "📦"
type: "tech"
topics: ["javascript", "typescript"]
published: false
---

# アロー関数でオブジェクトを返すときの `({})` って何者？

こんにちは、トラマトです。
Gemini君に書かせた[自分のホームページ](https://toramutton.me)の記事の処理コードを確認してたら、こんな書き方に遭遇しました。

```js
const localPosts = localPostsRaw.map((post) => ({
    title: post.data.title,
    url: `/blog/${post.id}/`,
    date: post.data.pubDate,
    image: post.data.heroImage,
    tags: post.data.tags,
    source: "local" as const,
}));
```

`({`とか`}));` って何これ、括弧が二重三重になってるの本当に見にくい！
…となっていたのですが、調べて自分なりに理解したので整理していきます。

---

## まず最初に：アロー関数の省略記法

アロー関数には、「中身が1つの**式**だけなら `{}` と `return` を省略できる」というルールがあります。

```js
// 普通の書き方
const double = (x) => { return x * 2; };

// 省略した書き方(同じ意味)
const double = (x) => x * 2;
```

`x * 2` は値として評価できる**式**なので、そのまま `return` してくれます。便利〜

---

## 問題：オブジェクトを返したいとき

じゃあオブジェクトを同じように省略しようとすると…？

```js
// ❌ これはアウト！
const fn = (x) => { title: x };
```

これだとJavaScript君が困っちゃうみたいで、`{ }` を見た瞬間、「あ、関数の本体(ブロック)だ！」と思われちゃう😢
`title: x` は `title` というラベルが付いた**文**として解釈されて、`return` もされません。

---

## 解決策：`( )` で包む

```js
// ✅ これが正解！
const fn = (x) => ({ title: x });
```

`( )` で包むことで、JavaScript君はこう判断するんですよ。

> 🦏「括弧の中にあるってことは、これは**式**だ！
じゃあ省略ルールを適用して、このオブジェクトをそのまま `return` するぞ〜」

つまり `({ })` は、

1. `( )` → 「これは式やで」という合図
2. `{ }` → オブジェクトリテラル本体

の組み合わせ。

---

## 最初のコードに戻ると

```js
// 省略…なし(同じ動き)
const localPosts = localPostsRaw.map((post) => {
    return {
        title: post.data.title,
        url: `/blog/${post.id}/`,
        date: post.data.pubDate,
        image: post.data.heroImage,
        tags: post.data.tags,
        source: "local" as const,
    };
});
```

```js
// 省略…あり(最初のコード)
const localPosts = localPostsRaw.map((post) => ({
    title: post.data.title,
    url: `/blog/${post.id}/`,
    date: post.data.pubDate,
    image: post.data.heroImage,
    tags: post.data.tags,
    source: "local" as const,
}));
```

動きは完全に同じです。`.map()` がオブジェクトを受け取って配列を作り、`localPosts` に入る処理です。
`return` がない分見やすいコードにできてますね。

---

## まとめ

| 書き方 | JavaScript君の解釈 |
|---|---|
| `=> { }` | 関数の本体(ブロック文)|
| `=> ({ })` | 式(オブジェクトを返す)|

つまり、あの二重三重括弧は **「`return` って書くのをサボりつつ、JavaScript君に勘違いさせないための書き方」** だったんです。

見にくいな〜と思ってたけど、ちゃんと意味があったんですね。勉強になりました。

---

*実際のコードは[Github](https://github.com/ToraMutton/toramutton-homepage)で公開しています！*
