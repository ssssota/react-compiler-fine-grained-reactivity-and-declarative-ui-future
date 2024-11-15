---
# You can also start simply with 'default'
theme: ohkime
# some information about your slides (markdown enabled)
title: React CompilerとFine Grained Reactivityと宣言的UIのこれから
info: |
  JSConfJP 2024
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
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

<h1>React Compilerと<br/>Fine Grained Reactivityと<br/>宣言的UIのこれから</h1>

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
{ "x": "ssssotaro", "github": "ssssota" }
```

今日喋る技術、React/Vue.js/Svelteいずれかの技術に強いこだわりはありません。
<span v-click>が、Svelte界隈に出没しがちです。</span>

どれが良い/悪いという話はしません。みんな違ってみんないい。

<style>
ruby {
  font-size: 3rem;
}
rt {
  font-size: 1rem;
}
</style>

---

## 宣言的UIと仮想DOMのはなし

宣言的UIはWeb開発の標準となり、Webだけでなくモバイルアプリケーションやデスクトップアプリケーションにも広がりを見せている。
いつの間に...？

...

10年前、我々は**仮想DOM**という概念に魂を震えさせていた。

（仮想DOMは**宣言的UIを実現するための手段**で宣言的UIそのものではないが）  
宣言的UIをここまで広めたのはReactやVue.jsのような仮想DOMを使ったライブラリの存在といっても過言ではない。

宣言的UIのこれからを語る上で、仮想DOMをまずは振り返る。

---

## 仮想DOMとはなんであったか

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
clicks: 5
---

## 仮想DOMのしくみ

<div class="grid grid-cols-2 gap-1">
  <ClickedOutline :click="1" class="m-0">
    0. 初回レンダリング時の仮想DOM
    <Excalidraw drawFilePath="./vdom.before.excalidraw" class="h-fit" />
  </ClickedOutline>

  <ClickedOutline :click="2" class="m-0">
    1. 状態変化時 仮想DOMを再構築する
    <Excalidraw drawFilePath="./vdom.after.excalidraw" class="h-fit" />
  </ClickedOutline>
</div>

<ClickedOutline :click="3">
2. 仮想DOMが構築できたら、差分を検出する (reconciliation / diffing)
</ClickedOutline>
<Arrow v-click="4" x1="230" y1="355" x2="500" y2="355" width="5" two-way color="#d21" />
<ClickedOutline :click="5">
3. 検出した差分をもとに、実際のDOMに反映する (render, commit)
</ClickedOutline>

---

## Virtual DOM is pure overhead

「仮想DOMは純粋なオーバーヘッドである」という意見。

Svelte作者のRich Harris氏が6年前に公開したブログ。
ことの発端はReactが速いという当時の主張。

Svelteは当時からSvelteコンポーネントをコンパイルすることで、仮想DOMを使わない宣言的UIを実現していた。

状態が変化時に通知する  
→変化箇所を特定(dirty check)  
→変化箇所に紐づくDOMを更新

---

## SolidJS

3年前、SolidJSが登場。Reactと同じくJSXを採用しながら、仮想DOMを使わない宣言的UIを実現。

状態は全てSignalで管理。Signalの値が変化すると、それを検知して該当のSignalが使われたDOMを更新。

これがいわゆる**Fine Grained Reactivity**。

> Fine-grained: きめの細かい

---

## Signal / Signals

Fine Grained ReactivityのベースにあるのがSignal。SignalはStreamやObservableのような概念で、単一の値を持ちその値が変化すると通知できる。

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
// Vue.js Composition API
const count = ref(0);
const message = ref("");
watchEffect(() => {
  console.log(count.value, message.value);
});
```
````

---

## Svelte 5 / Vue Vapor

いずれもSignalを導入。

SolidJSとは異なりDSLを提供しているため、SolidJSのような直感に反する制約を軽減している、とも言える。

---

## Reactの答え

Virtual DOM is pure overhead に対するReactの答えが、React Compiler。

仮想DOMのオーバーヘッドである、差分検出、仮想DOMツリー構築をコンパイラによりメモ化することで極限までオーバーヘッドを減らす。

---

## React Compilerの課題(?)

React Compilerは、開発者にReactのルールを強制する。

これに対し、JavaScriptの意味論が変わってしまうという意見もある。

---

## Million.js

ReactのReconcilerを差し替え、dirty checkにすることで、高速化を図っていた。

現在はLinterに注力されている模様。(新しいドキュメントではLinterのみが紹介されている)
