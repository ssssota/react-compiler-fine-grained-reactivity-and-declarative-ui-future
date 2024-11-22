---
# You can also start simply with 'default'
theme: ohkime
# some information about your slides (markdown enabled)
title: React CompilerとFine-Grained Reactivityと宣言的UIのこれから
info: |
  JSConfJP 2024
# https://sli.dev/features/drawing
drawings:
  persist: false
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# take snapshot for each slide in the overview
overviewSnapshots: true
fonts:
  sans: BIZ UDPGothic, Zen Kaku Gothic New
  serif: BIZ UDPMincho
  # default
  weights: "200,400,700"
---

<h1>React Compilerと<br/>Fine-Grained Reactivityと<br/>宣言的UIのこれから</h1>

JSConf JP 2024 - TOMIKAWA Sotaro (ssssota)

---

## はじめまして！

<ruby>冨川<rp>(</rp><rt>TOMIKAWA</rt><rp>)</rp></ruby>&emsp;<ruby>宗太郎<rp>(</rp><rt>Sotaro</rt><rp>)</rp></ruby>

株式会社ZOZO フロントエンドエンジニア（テックリード）

```json
{
  "x": "ssssotaro",
  "bsky": "ssssota.bsky.social",
  "github": "ssssota"
}
```

仕事ではReact、趣味はSvelteかPreactを使っていることが多いです。

![](https://github.com/ssssota.png)

<style>
ruby {
  font-size: 3rem;
}
rt {
  font-size: 1rem;
}
img {
  position: absolute;
  right: 5rem;
  top: 5rem;
  width: 10rem;
  height: 10rem;
}
</style>

---

## きょうは宣言的UIのはなし

宣言的UIはWeb開発の標準となり、Webだけでなくモバイルアプリケーションやデスクトップアプリケーションにも広がりを見せている。

- Web: React, Vue.js, Svelte, Preact, etc...
- モバイル: SwiftUI, Jetpack Compose, React Native, Flutter, etc...

<v-click>

APIはそれぞれ少しずつ異なるものの、  
**「状態をもとにUIを宣言する」** という基本的な考え方は共通している。

今日はWeb開発における宣言的UIのこれまでとこれからを考える。

</v-click>

---
layout: cover
---

# １章 宣言的UIと仮想DOM

---

## 宣言的UIと仮想DOM

10年前、“**仮想DOM**という概念が俺達の魂を震えさせ”ていた。

（仮想DOMは**宣言的UIを実現するための手段**で宣言的UIそのものではないが）  
宣言的UIをここまで広めたのはReactやVue.jsのような仮想DOMを使ったライブラリの存在といっても過言ではない。

宣言的UIのこれからを語る上で、仮想DOMをまずは振り返る。

[*1]: https://qiita.com/mizchi/items/4d25bc26def1719d52e6

---

## 仮想DOMのしくみ

```jsx
function App({ name }) {
  return (
    <div>
      <h1>Hello {name}!</h1>
      <p>Welcome to the session.</p>
    </div>
  );
}
```

例えばこんなコンポーネントを考える。

---

## 仮想DOMのしくみ

<div class="grid grid-cols-2 gap-1">

<div
  v-click="1"
  class="outline-blue-500/50 outline-4 m-0"
  :class="{outline: $clicks === 1 && $renderContext === 'slide'}"
>

0\. 初回レンダリング時の仮想DOM

<Excalidraw drawFilePath="./vdom.before.excalidraw" class="h-fit" />

</div>

<div
  v-click="2"
  class="outline-blue-500/50 outline-4 m-0"
  :class="{outline: $clicks === 2 && $renderContext === 'slide'}"
>

1\. 状態変化時 仮想DOMを再構築する

<Excalidraw drawFilePath="./vdom.after.excalidraw" class="h-fit" />

</div>

</div>

<div
  v-click="3"
  class="outline-blue-500/50 outline-4 m-0"
  :class="{outline: $clicks === 3 && $renderContext === 'slide'}"
>

2\. 仮想DOMが構築できたら、差分を検出する (reconciliation / diffing)

</div>
<Arrow v-click="4" x1="230" y1="370" x2="500" y2="370" width="5" two-way color="#d21" />
<div
  v-click="5"
  class="outline-blue-500/50 outline-4 m-0"
  :class="{outline: $clicks === 5 && $renderContext === 'slide'}"
>

3\. 検出した差分をもとに、実際のDOMに反映する (render, commit)

</div>

---

## Virtual DOM is pure overhead

「仮想DOMは純粋なオーバーヘッドである」  
Svelte作者のRich Harris氏が6年前に公開したブログのタイトル。  
今後の宣言的UIを考える上での重要なキーワード。

<v-clicks>

1. 仮想DOMの差分検出自体コストがかかる
   - 仮想DOMツリーを探索して、効率よく実際のDOMに適用するための差分を検出する必要がある
   - Reactでは $O(n)$ のアルゴリズムを使っているとされる(コスト小)
2. 仮想DOMの構築自体コストがかかる
   - 仮想DOMツリーの構築では様々な計算やアロケーションが何度も発生する
     - 各種配列、仮想DOM自体のオブジェクト、インライン関数、、、

</v-clicks>

[*1]: https://svelte.dev/blog/virtual-dom-is-pure-overhead
[*2]: https://ja.legacy.reactjs.org/docs/reconciliation.html#motivation

---

## React Compiler

今年春発表されたReact Compiler (React Forgetは2021年発表)。  
これが、先の問題を解決する。

> 2. 仮想DOMの構築自体コストがかかる
>    - 仮想DOMツリーの構築では様々な計算やアロケーションが何度も発生する
>      - 各種配列、仮想DOM自体のオブジェクト、インライン関数、、、

<v-click>

*様々な計算やアロケーション*とは？どのように解決するのか？

</v-click>

[*1]: https://ja.react.dev/blog/2021/12/17/react-conf-2021-recap#react-without-memo

---

## 例：仮想DOMオブジェクトの計算・アロケーションコスト削減

<div class="grid grid-cols-2 gap-1">

```jsx
function App({ name }) {
  return <h1>Hello {name}!</h1>;
}
```

<a v-click href="https://playground.react.dev/#N4Igzg9grgTgxgUxALhAMygOzgFwJYSYAEAggA5kAUwRmAhgLYJEC+AlEcADrFEwI5YxADwALAIwA+ABIIANnIid6TFgEJhAegmSA3DxYgWQA" target="_blank" rel="noreferrer">

```jsx
function App(t0) {
  const $ = _c(2);
  const { name } = t0;
  let t1;
  if ($[0] !== name) {
    t1 = <h1>Hello {name}!</h1>;
    $[0] = name;
    $[1] = t1;
  } else {
    t1 = $[1]; // nameが同じ→再利用
  }
  return t1;
}
```

</a>

</div>

---

## 例：関数の計算・アロケーションコスト削減

<div class="grid grid-cols-2 gap-1">

```jsx
function List({ items }) {
  return (
    <ul>
      {items.map((item) => {
        return <li>{item}</li>;
      })}
    </ul>
  );
}
```

<a v-click href="https://playground.react.dev/#N4Igzg9grgTgxgUxALhAMygOzgFwJYSYAEAMnmDgBTBF44IC2YRAvgJRHAA6xRMCOWMUo8iYogB4oAGwB8o8WOB1GYAHQMAhgAdKlFQw4BeWZwWKx-QTGITpeWcvoMWEgPT3ZAbnPj2LX3cZeV42H0wWEBYgA" target="_blank" rel="noreferrer">

```jsx
function List(t0) {
  const $ = _c(4);
  const { items } = t0;
  let t1;
  if ($[0] !== items) {
    t1 = items.map(_temp);
  // . . .
}
// ↓トップレベルへ移動
function _temp(item) {
  return <li>{item}</li>;
}
```

</a>

</div>

---

## React Compiler

`useMemo` や `React.memo` を使えばできなくもないが、
開発者がそれを意識しなければならなかった。

これらのコストをReact Compilerが最適化する。

1. 仮想DOMオブジェクトをキャッシュする
2. インライン関数をトップレベルに移動する/キャッシュする

<v-click>

状態変化時、通常は状態が変化したコンポーネントの子孫も再構築されるが、  
React Compilerでは再構築されるコンポーネントを最小限に抑える。

</v-click>

---

## 仮想DOMのしくみ（with React Compiler）

<div class="grid grid-cols-2 gap-1">

<div>

0\. 初回レンダリング時の仮想DOM

<Excalidraw drawFilePath="./vdom.before.excalidraw" class="h-fit" />

</div>

<div
  v-click="1"
  class="outline-blue-500/50 outline-4 m-0"
  :class="{outline: $clicks === 1 && $renderContext === 'slide'}"
>

1\. 状態変化時 仮想DOMを再構築する

<Excalidraw drawFilePath="./vdom.after.compiler.excalidraw" class="h-fit" />

</div>

</div>

<div v-click="2">

2\. 仮想DOMが構築できたら、差分を検出する (reconciliation / diffing)

</div>
<Arrow v-click="3" x1="230" y1="370" x2="500" y2="370" width="5" two-way color="#d21" />
<div v-click="4">

3\. 検出した差分をもとに、実際のDOMに反映する (render, commit)

</div>

---

## 閑話休題 宣言的UIとJSX

いまでは様々なフレームワークが利用しているJSX。  
当初はFacebook(現Meta)がReactのために開発した言語拡張。

JSXはReactとともに普及し、  
現在ではマークアップのデファクトスタンダードとなっている。

独自文法とは異なり周辺ツールチェーンの恩恵を受けやすいのも大きなメリット。

- Parser
- Linter / Formatter
- Transformer
- etc...

---

## 宣言的UIと仮想DOMとReact Compiler

ここまで、**ReactのReactによるReactのためのReact Compiler**を使った _Virtual DOM is pure overhead_ への対応を見てきた。
これは仮想DOMと共存する道の1つと言える。

(仮想DOMのVue.js SFCもコンパイルを伴うため最適化が行われている)

一方で、仮想DOMを使わない宣言的UIも存在する。

[*]: https://vuejs.org/guide/extras/rendering-mechanism.html#compiler-informed-virtual-dom

---
layout: cover
---

# ２章 Fine-Grained Reactivity

---

## SolidJS

3年前、SolidJS(v1)が登場。Reactと同じくJSXを採用しながら、仮想DOMを使わない宣言的UIを実現。

状態は全てSignalsで管理。Signalsの値が変化すると、それを検知して該当のSignalsが使われたDOMを更新。

<v-click>

これがいわゆる**Fine-Grained Reactivity**。

> Fine-grained: きめの細かい

</v-click>

---

## Signals

Fine-Grained Reactivityのベースにあるのが**Signals**。SignalsはStreamやObservableのような概念で、単一の値を持ちその値が変化すると通知できる。

StreamやObservableではなく、Signalsが宣言的UIで重宝されるのはインターフェースと柔軟性のバランスが取れているため。

---

## Signals

<div class="grid grid-cols-[auto_auto] grid-rows-2 gap-1">

これらは状態を購読する例。

<v-click>

```jsx
// React Hooks
const [count, setCount] = useState(0);
useEffect(() => {
  console.log(count);
}, [count]);
```

</v-click>
<v-click>

```js
// Svelte (svelte/store)
const count = writable(0);
count.subscribe((value) => {
  console.log(value);
});
```

</v-click>
<v-click>

```js
// Vue.js (@vue/reactivity)
const count = ref(0);
watchEffect(() => {
  console.log(count.value);
});
```

</v-click>
</div>

---

## SolidJSの弱点

SolidJSのコンポーネントは1度しか実行されない。

1度実行でコンポーネントのすべての状態を返す必要がある(イメージ)。

また、それゆえの制約がある。

---

## SolidJSの弱点 - 制約１

算出プロパティを使う時は関数にする

<a href="https://playground.solidjs.com/anonymous/904d4049-de74-4929-96c2-5dc1d4dea9ef" target="_blank" rel="noreferrer">

<!-- prettier-ignore -->
````md magic-move
```jsx
function App() {
  const [count, setCount] = createSignal(0);
  const double = count() * 2;
  return (
    <>
      <p>{count()} * 2 = {double}</p>
      <button onClick={() => setCount((c) => c + 1)}> +1 </button>
    </>
  );
}
```

```jsx
function App() {
  const [count, setCount] = createSignal(0);
  const double = () => count() * 2;
  return (
    <>
      <p>{count()} * 2 = {double()}</p>
      <button onClick={() => setCount((c) => c + 1)}> +1 </button>
    </>
  );
}
```
````

</a>

---

## SolidJSの弱点 - 制約２

早期リターンができない

<a href="https://playground.solidjs.com/anonymous/128721e1-e995-4df9-b3f4-7355ff7db176" target="_blank" rel="noreferrer">

````md magic-move
```jsx
function App() {
  const [count, setCount] = createSignal(0);
  if (count() === 0) {
    return <button onClick={() => setCount(1)}>Start!</button>;
  }
  return <p>Count is {count()}</p>;
}
```

```jsx
function App() {
  const [count, setCount] = createSignal(0);
  return (
    <Show when={count() === 0} fallback={<p>Count is {count()}</p>}>
      <button onClick={() => setCount(1)}>Start!</button>
    </Show>
  );
}
```
````

</a>

---

## Svelte 5 と Vue Vapor

いずれもFine-Grained Reactivityを実現。(Svelte 5は10月リリース、Vue VaporはWIP)

SolidJSとは異なり独自の文法を提供しているため、「SolidJSのような制約を軽減している」とも言える。
そもそもリターンを書かないから早期リターンもない。

一方で、JSXではない故の問題もある。  
最近はRust製の高速なツールチェーンが登場しているが、対応が後回しになりがち。

---

## SolidJS と Svelte 5 と Vue Vapor

いずれも開発者が書いたコードがコンパイルされ、関数コンポーネントになる。  
この関数コンポーネントは実際のDOMを返す(Svelteは若干異なるが省略)。

<div class="flex justify-between items-start">

<!-- prettier-ignore -->
```vue
<!-- コンパイル前 -->
<script setup>
defineProps(["name"]);
</script>
<template>
  <h1>
    Hello {{ name }}!
  </h1>
</template>
```

<a v-click href="https://vapor-repl.netlify.app/#__VAPOR__eNp9UF1LwzAU/SsxPnSD0SL6NOtAZaA+6FDxxfhQ2rsts/kgSWul9L97k26z4lihND3n3JtzTkuvtY7rCuiUpjY3XDtiwVV6xmQBSy5hYZS2o3dGZSaA0Y/xJZNp0ktRlDoQuswc4JmQdH0WvoTcQVkq0rbEj5GuOwl0Evg0GQzRCeV4VROvnSgHLty3hitGhSqqEu/drvUPF1oZR1qSG8Adbxn+YgrSkaVRgkSYJqk9GP2b8bJeFCfb4APR330jfMexUJV0o+g00zoa73OnBa8JL9Af4mguTRCYYRRncyWXfBVvrJKYpvXbGc2V0LwE86QdV9IyOiWB8VyGRX09BMyZCiY7PF9D/nkA39jGY4wuDFgwNZaz51xmVuB6ev7yCA2e9+SuyiPkM1hVVt5jL7upZIG2B7rg9j60yeXq1c4bB9LuQnmjXtkFPaPY7+2R6L92z+OLMMdkR7sfaXDXKA==" target="_blank" rel="noreferrer">

<!-- prettier-ignore -->
```js
// コンパイル後 (一部手で調整)
const t0 = _template("<h1></h1>")
function render(_ctx, $props) {
  const n0 = t0()
  _renderEffect(() => {
    _setText(n0, "Hello ", $props.name, "!")
  })
  return n0
}
```

</a>

</div>

---

## Fine-Grained Reactivity

_Virtual DOM is pure overhead_ に対する1つの答えが  
**Fine-Grained Reactivity**。

そもそも仮想DOMを使わなければ、仮想DOMのオーバーヘッドはなくなる。

オブジェクトや関数のアロケーションも、関数コンポーネント自体は一度しか呼ばれないので問題にならない。

主なプレイヤーは**SolidJS**、**Svelte 5**、**Vue Vapor**。  
（Vue Vaporは仮想DOMモードとの併用も可能）

---

## 閑話休題 Signals

Fine-Grained Reactivityの基本はSignals。  
このSignal、各フレームワークが独自に実装している。当然互換性はない。

そこで、TC39のプロポーザルとしてSignals標準化の動きがある。(Stage 1)

現状すぐに使えるわけではないが、標準化されれば各Fine-Grained Reactivity系フレームワークの互換性が向上する可能性もある。
（パフォーマンスはそれほど変わらない）

---

## 2.7章 いまとこれから

仮想DOMが宣言的UIを広めてきたが、仮想DOMを使わない宣言的UIも勢力を拡大している。

Reactは仮想DOM由来の問題を解決するためにReact Compilerを開発中。  
React Compilerは名前の通りReact(のコンポーネント)をコンパイルする。

他のライブラリはSignalを使ったFine-Grained Reactivityがアツい。
Fine-Grained Reactivityでは仮想DOMを使わず、状態に追従する実際のDOM要素を作り出す。

---

## これからの宣言的UIに必要な要素

いまを考えると、これからの宣言的UIには次のような要素が求められると考えられる。

<v-clicks>

1. パフォーマンス
   - 命令型のコード(=フレームワークなし)に漸近するスピード
2. 開発体験
   - 開発者が違和感のないコードを書けること
   - 外部ツールとの親和性
3. 互換性
   - 既存コードとの親和性

</v-clicks>

---

## 宣言的UIと仮想DOMとFine-Grained Reactivity

宣言的UIはWeb開発の標準となった。

React CompilerやFine-Grained Reactivityは、  
これからの宣言的UIのキモになるかもしれない。

<v-click>

一方で、仮想DOMが極端に**遅いわけではない**。  
我々は現実的なスピードで動作するWebアプリを作っているし、使えている。

</v-click>

<v-click>

新しい技術を注視しつつ、いまある仮想DOMなどの技術と課題を見つめていけばよい。

</v-click>

---

# まとめ

- 宣言的UIはWebのものではなく、広く使われるようになっている
- 仮想DOMは宣言的UIを広めたが、オーバーヘッドも問題視されている
- React CompilerはReactの仮想DOMオーバーヘッドを解決する
- Fine-Grained Reactivityは仮想DOMを使わず、宣言的UIを実現する
- これからの宣言的UIには高いパフォーマンス・開発体験・互換性が求められる
- 仮想DOMも死なないので引き続き魂を震わせてOK
