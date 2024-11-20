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

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #365d62 10%, #146b8c 50%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: bold;
  font-family: 'Zen Kaku Gothic New';
  line-height: 1.2 !important;
}
</style>

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

<style>
ruby {
  font-size: 3rem;
}
rt {
  font-size: 1rem;
}
</style>

---

## 宣言的UIのはなし

宣言的UIはWeb開発の標準となり、Webだけでなくモバイルアプリケーションやデスクトップアプリケーションにも広がりを見せている。

- Web: React, Vue.js, Svelte, etc...
- モバイル: React Native, Flutter, SwiftUI, Jetpack Compose, etc...

APIはそれぞれ少しずつ異なるものの、  
**「状態をもとにUIを宣言する」** という基本的な考え方は共通している。

今日はWeb開発における宣言的UIのこれまでとこれからを考える。

---

## 宣言的UIと仮想DOM

10年前、我々は**仮想DOM**という概念に魂を震えさせていた。

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

Svelte作者のRich Harris氏が6年前に公開したブログ。  
今後の宣言的UIを考える上でのキーワード。仮想DOMは純粋なオーバーヘッド。

<v-clicks>

1. 仮想DOMの差分検出自体コストがかかる
   - 仮想DOMツリーを探索して、効率よく実際のDOMに適用するための差分を検出する必要がある
   - Reactでは $O(n)$ のアルゴリズムを使っているとされる(コスト小)
2. 仮想DOMの構築自体コストがかかる
   - 仮想DOMツリーの構築では何度も様々なアロケーションが発生する
     - 各種配列、仮想DOM自体のオブジェクト、インライン関数、、、

</v-clicks>

[*1]: https://svelte.dev/blog/virtual-dom-is-pure-overhead
[*2]: https://ja.legacy.reactjs.org/docs/reconciliation.html#motivation

---

## React Compiler

React Conf 2021で発表されたReact Compiler (当時React Forget)。  
これが、先の問題を解決する。

> 2. 仮想DOMの構築自体コストがかかる
>    - 仮想DOMツリーの構築では何度も様々なアロケーションが発生する
>      - 各種配列、仮想DOM自体のオブジェクト、インライン関数、、、

<v-click>

様々なアロケーションとは？どのように解決するのか？

</v-click>

[*1]: https://ja.react.dev/blog/2021/12/17/react-conf-2021-recap#react-without-memo

---

## 例：仮想DOMオブジェクトのアロケーションコスト削減

<div class="grid grid-cols-2 gap-1">

```jsx
function App({ name }) {
  return <h1>Hello {name}!</h1>;
}
```

<v-click>

<a href="https://playground.react.dev/#N4Igzg9grgTgxgUxALhAMygOzgFwJYSYAEAggA5kAUwRmAhgLYJEC+AlEcADrFEwI5YxADwALAIwA+ABIIANnIid6TFgEJhAegmSA3DxYgWQA" target="_blank" rel="noreferrer">

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

</v-click>

</div>

---

## 例：関数のアロケーションコスト削減

<div class="grid grid-cols-2 gap-1">

```jsx
function List({ items }) {
  return (
    <ul>
      {item.map((item) => {
        return <li>{item}</li>;
      })}
    </ul>
  );
}
```

<v-click>

<a href="https://playground.react.dev/#N4Igzg9grgTgxgUxALhAMygOzgFwJYSYAEAMnmDgBTBF44IC2YRAvgJRHAA6xRMCOWMUo8iYogB4oAGwB8o8WOB1GAOgYBDAA6VKKhhwC8szgsVj+gmMQnS8s5fQYsJAejuyA3GfHsWPtxl5XjZvTBYQFiA" target="_blank" rel="noreferrer">

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

</v-click>

</div>

---

## React Compiler

我々開発者がカジュアルに書いたインライン関数やオブジェクトは、  
仮想DOM構築時にアロケーションされる。

それらをReact Compilerが最適化する。

1. 仮想DOMオブジェクトをキャッシュする
2. インライン関数をトップレベルに移動する/キャッシュする

状態変化時、通常は状態が変化したコンポーネントの子孫も再構築されるが、  
React Compilerでは再構築されるコンポーネントを最小限に抑える。

Reactのルールに従っていないコンポーネントは矯正する(ESLintプラグイン)。

---

## JSXというメリット

いまでは様々なフレームワークが利用しているJSX。  
当初はFacebook(現Meta)がReactのために開発した言語拡張。

JSXはReactとともに普及し、  
現在ではマークアップのデファクトスタンダードとなっている。

これにより、周辺ツールチェーンの恩恵を受けやすいのも大きなメリット。

- Parser
- Linter / Formatter
- Transformer
- etc...

---

## SolidJS

3年前、SolidJS(v1)が登場。Reactと同じくJSXを採用しながら、仮想DOMを使わない宣言的UIを実現。

状態は全てSignalで管理。Signalの値が変化すると、それを検知して該当のSignalが使われたDOMを更新。

<v-click>

これがいわゆる**Fine-Grained Reactivity**。

> Fine-grained: きめの細かい

</v-click>

---

## Signal / Signals

Fine-Grained Reactivityのベースにあるのが**Signal**。SignalはStreamやObservableのような概念で、単一の値を持ちその値が変化すると通知できる。

StreamやObservableではなく、Signalが宣言的UIで重宝されるのはインターフェースと柔軟性のバランスが取れているため。

暗黙的な購読は特徴的。以下は複数の状態を購読する例。

````md magic-move
```js
// React Hooks
const [count, setCount] = useState(0);
const [message, setMessage] = useState("");
useEffect(() => {
  console.log(count, message);
}, [count, message]);
```

```js
// Observable (RxJS)
const count$ = new Subject(0);
const message$ = new Subject("");
zip(count$, message$).subscribe(([count, message]) => {
  console.log(count, message);
});
```

```js
// Vue.js Composition API (=Signal)
const count = ref(0);
const message = ref("");
watchEffect(() => {
  console.log(count.value, message.value);
});
```
````

---

## SolidJSの弱点

SolidJSのコンポーネントは1度しか実行されない。その1度でコンポーネントのすべての状態を返す必要がある(イメージ)。
また、それゆえの制約がある。

````md magic-move
```jsx
// 早期リターンができない
function App() {
  const [count, setCount] = createSignal(0);
  if (count() === 0) {
    return <button onClick={() => setCount(1)}>Start!</button>;
  }
  return <p>Count is {count()}</p>;
}
```

```jsx
// 早期リターンができない
function App() {
  const [count, setCount] = createSignal(0);
  return (
    <Show
      when={count() !== 0}
      fallback={<button onClick={() => setCount(1)}>Start!</button>}
    >
      <p>Count is {count()}</p>
    </Show>
  );
}
```

```jsx
// 算出プロパティを使う時は関数にする
function App() {
  const [count, setCount] = createSignal(0);
  const double = count() * 2;
  return (
    <p>
      {count()} * 2 = {double}
    </p>
  );
}
```

```jsx
// 算出プロパティを使う時は関数にする
function App() {
  const [count, setCount] = createSignal(0);
  const double = () => count() * 2;
  return (
    <p>
      {count()} * 2 = {double()}
    </p>
  );
}
```
````

---

## Svelte 5 と Vue Vapor

いずれもFine-Grained Reactivityを実現。(Svelte 5は10月リリース、Vue VaporはWIP)

SolidJSとは異なり独自の文法を提供しているため、「SolidJSのような制約を軽減している」とも言える。そもそもリターンを書かないから早期リターンもない。

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

<v-click>

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

</v-click>

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

## 群雄割拠 Signals

Fine-Grained Reactivityの基本はSignals。  
このSignal、各フレームワークが独自に実装している。当然互換性はない。

そこで、TC39のプロポーザルとしてSignals標準化の動きがある。(Stage 1)

現状すぐに使えるわけではないが、標準化されれば各Fine-Grained Reactivity系フレームワークの互換性が向上する可能性もある。

---

## ここまでのまとめ

仮想DOMが宣言的UIを広めてきたが、仮想DOMのオーバーヘッドは否めない。

Reactはこの問題を解決するためにReact Compilerを開発中。React Compilerは名前の通りReact(のコンポーネント)をコンパイルする。

他のライブラリはSignalを使ったFine-Grained Reactivityに注力している。
Fine-Grained Reactivityでは仮想DOMを使わず、状態に追従する実際のDOM要素を作り出す。

---

## 共通点

Fine-Grained ReactivityとReact Compilerは共通点がある。
それは、開発者が書いたコードを変換(・コンパイル)して最適化するという点。

_(TypeScriptやJSXは大前提として省略して)_

<v-clicks>

- React CompilerはReactコンポーネントをコンパイルして最適化する
- Fine-Grained ReactivityはSignalを使ったコンポーネントを  
  変換して実際のDOMを返す関数に変換する

</v-clicks>

---

## これからの宣言的UIに必要な要素

<v-clicks>

1. パフォーマンス
   - 命令型のコード(=フレームワークなし)に漸近するスピード
2. 開発体験
   - 開発者が違和感のないコードを書けること
   - 外部ツールとの親和性

</v-clicks>

---

## 宣言的UIのこれから

最初にも述べた通り宣言的UIは **「状態をもとにUIを宣言する」**。  
いかに状態を作り、それをUIに反映するかということ。

ただ仮想DOMを使う単純に仮想DOMを使うのではなく、
ReactはReact Compilerを使いつつも仮想DOMを使い続けるし、  
それ以外はFine-Grained Reactivityに注力する。

一方で、仮想DOMが極端に**遅いわけではない**。  
我々は現実的なスピードで動作するWebアプリを作っているし、使えている。

新しいFine-Grained Reactivityという選択肢を頭に入れつつ、  
それぞれの進化に注目していきたい。

---
