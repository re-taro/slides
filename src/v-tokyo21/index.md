---
marp: true
paginate: true
theme: re-taro
image: https://slides.re-taro.dev/v-tokyo21/index.png
header: Can oxc be the next generation JS toolchain development platform?
footer: 2024-07-26 | Vue.js v-tokyo Meetup #21
---

# Can oxc be the next generation JS toolchain development platform?

## りんたろー / re-taro

---

<!--
## 自己紹介

りんたろーです。

学生してます。緑の会社で Web エンジニアもしてます。

初めて登壇するのでガチガチに緊張していますが、よろしくお願いします。
-->

# Self introduction

![bg 50% right:50%](images/re-taro.png)

## りんたろー / re-taro

- Student / Web Engineer
- TypeScript / Rust
- [re-taro.dev](https://re-taro.dev)

---

<!--
## 今までの JS ツールチェイン

突然ですが、これまでの私達の開発を支えてきたツールチェインを振り返ってみましょう。

- Webpack
- Babel
- ESLint
- Prettier

これらのツールに共通している点はなんでしょう？それは...
-->

# JS toolchain so far

- Webpack
- Babel
- ESLint
- Prettier

What do these tools have in common?

---

<!--
**JavaScript(TypeScript)で実装されているという点**
-->

## All implemented in JavaScript(TypeScript)

---

<!--
## 昨今の JS ツールチェイン

では、昨今登場しているJSツールチェインを見てみましょう。

- Vite
  - esbuild
- swc
- Rolldown
- Biome
- Oxc

これらのツールに共通している点はなんでしょう？それは...
-->

# Recent JS toolchain

- Vite
  - esbuild
- swc
- Rolldown
- Biome
- Oxc

What do these tools have in common?

---

<!--
**RustやGoで実装されているという点**
-->

## All implemented in Rust or Go

---

<!--
## なぜ Rust で書かれることが多いのだろう？

前提として、私は Golang が書けない読めない人なので、 Golang については語れません。

Rust が JS ツールチェインの実装言語として選ばれる理由はいくつか考えられます。

- ライフタイムアノテーション
- ゼロコスト抽象化

これらの言語仕様によって、開発者はアルゴリズムの複雑さにのみ注意するだけでよくなるため、より高速なツールチェインを簡単に実装することができます。
-->

# Why Rust?

- Lifetime annotation
- Zero-cost abstraction

---

<!--
それでは今日の本題です。

## Oxc ってなんでしょう？
-->

# What is Oxc?

---

<!--
Oxc とは、パーサー、リンター、フォーマッタ、トランスパイラ、ミニファイア、リゾルバなど、JavaScriptに不可欠なコンパイラツールのコレクションです。

また、Rolldownなどの次世代のツールチェインのサポートをしているのもOxcです。
-->

# What is Oxc?

- :rocket: High performance
- :toolbox: Constructing essential compiler tools for JS
  - Parser
  - Transpiler
  - Resolver
  - ...and more!!
- :handshake: Supporting next-generation toolchains like Rolldown

---

<!--
## なぜ Oxc なのか？

では次に Oxc の思想についてです。

Oxc では以下のような思想で開発されています。

- パフォーマンス
- 拡張性
- 信頼性
 -->

# Why Oxc?

- Performance
- Extensibility
- Reliability

---

<!--
それぞれ詳しく見ていきましょう。

まずはパフォーマンスです。
 -->

<style scoped>
  li:nth-child(1) {
    text-decoration: underline;
  }
  li:not(:nth-child(1)) {
    opacity: 0.4;
  }
</style>

# Why Oxc?

- Performance
- Extensibility
- Reliability

---

<!--
無駄のない設計により、Oxc は高速な処理を実現しています。

Oxc TypeScript + JSX トランスフォーマーは swc よりおよそ2倍高速です。
-->

<style scoped>
  .image > p {
    margin-bottom: 0;
  }
  .ref {
    font-size: 0.7em;
  }
</style>

## Performance

<div class="image">

![w:600](images/oxc_vs_swc.svg)

</div>

<div class="ref">

ref: https://github.com/oxc-project/bench-javascript-parser-written-in-rust

</div>

Lean architecture, Oxc achieves high-performance processing.

---

<!--
### ではなぜ Oxc は高速なのでしょうか？

ここでは Oxc が高速な理由について少し紹介します。

1. アリーナアロケーターを用いた AST の高速なメモリ割り当てと解放
  - `bumpalo` と呼ばれるライブラリを使用しています。これはバンプアロケーターを提供していて、定数時間でのメモリ割り当てを可能にし、メモリの断片化を減らします。
  - 以下のように AST にライフタイムアノテーションを付与することで AST node がアリーナに依存していることを明示でき、アリーナが解法されると AST node も解法されるようになります。
    ```rs
    pub enum Statement<'a> {
      Expression(ExpressionNode<'a>),
    }
    ```
2. `compact_str`を用いてインライン化された文字列
  - Rust の `std::string::String` の実態は常にヒープにアロケートされます。一方で `compact_str::CompactStr` は24バイトまでの文字列をスタックに配置することで、ヒープアロケーションを回避します。

Oxc のすごいところはこの２つ以外でヒープの確保が行われないことです。
-->

<style scopde>
  h2 {
    margin-bottom: 0;
  }
  h3 {
    margin-top: 0;
  }
</style>

## Performance

### Why is Oxc so fast?

1.  Fast memory allocation and deallocation of AST using arena allocator

    - Using [`bumpalo`](https://crates.io/crates/bumpalo)
    - Marking AST nodes with lifetime annotations to depend on the arena
      ```rs
      pub enum Statement<'a> {
        Expression(ExpressionNode<'a>),
      }
      ```

2.  Inline strings using [`compact_str`](https://crates.io/crates/compact_str)

    - `std::string::String` always allocates on the heap
    - `compact_str::CompactStr` places strings up to 24 bytes on the stack

Oxc does not allocate on the heap except for these two.

---

<!--
次に拡張性についてです。
 -->

<style scoped>
  li:nth-child(2) {
    text-decoration: underline;
  }
  li:not(:nth-child(2)) {
    opacity: 0.4;
  }
</style>

# Why Oxc?

- Performance
- Extensibility
- Reliability

---

<!--
次に拡張性についてです。

Oxc は最初から他のプロジェクトのコンポーネントとして使えるように設計されています。

以下に Oxc を使用しているプロジェクトをいくつか紹介します。

- Rolldown
  - https://github.com/rolldown/rolldown
- Biome
  - https://github.com/biomejs/biome
- tauri
  - https://github.com/tauri-apps/tauri
- kuma-ui
  - https://github.com/kuma-ui/kuma-ui
-->

## Extensibility

Oxc is designed to be used as a component of other projects from the beginning.

Here are some projects that use Oxc.

- Rolldown
  - https://github.com/rolldown/rolldown
- Biome
  - https://github.com/biomejs/biome
- tauri
  - https://github.com/tauri-apps/tauri
- kuma-ui
  - https://github.com/kuma-ui/kuma-ui

---

<!--
信頼性についても見ていきましょう。
-->

<style scoped>
  li:nth-child(3) {
    text-decoration: underline;
  }
  li:not(:nth-child(3)) {
    opacity: 0.4;
  }
</style>

# Why Oxc?

- Performance
- Extensibility
- Reliability

---

<!--
Oxc はコードの正しさと信頼性を保証するために、コード周りのインフラに重点を置いています。

Oxc では以下の結果を公開しています

- Test262, Babel, TypeScriptのテストスイートのパス率
- パフォーマンスベンチマーク
- パッケージサイズ
-->

## Reliability

Oxc focuses on the infrastructure around the code to ensure correctness and reliability.

- Test262, Babel, TypeScript test suite pass rate
- Performance benchmark
- Package size

---

<!--
特に注目すべきは oxc_parser のテストスイートのパス率です。

これらのデータは `oxc_parser`, `swc_ecma_parser`, `biome` のそれぞれのテストスイートのパス率を比較、まとめたものです。

| パーサー          | test262 | Babel   | TypeScript |
| --------------- | ------- | ------- | ---------- |
| oxc_parser      | 100%    | 90%     | 99%        |
| swc_ecma_parser | 54%     | No data | No data    |
| biome           | 97%     | 92%     | 76%        |

特徴的なのは、oxc_parser が 他の Rust 実装のパーサーに比べて高く、精密な結果を出していることです。
-->

## Reliability

### Of note is the pass rate of the `oxc_parser` test suite

| Parser          | test262 | Babel   | TypeScript |
| --------------- | ------- | ------- | ---------- |
| oxc_parser      | 100%    | 90%     | 99%        |
| swc_ecma_parser | 54%     | No data | No data    |
| biome           | 97%     | 92%     | 76%        |

---

<!--
## Oxc の未来

Oxc は"第3世代"の JS ツールチェインです。

先人たちのツールチェインを参考にしつつ、Oxc は次のような特徴を持っています。
- 最適な選択を基礎に取り込む
- edge bundling
- JS とのシームレスな相互運用性
- 無駄がない
  - swc や biome はアーキテクチャによる制約があったりコンパイラのユースケースを考慮されていない
-->

# Future of Oxc

Oxc is a "third-generation" JS toolchain.

- Incorporating the best choices
- Edge bundling
- Seamless interoperability with JS
- No waste
  - swc and biome have architectural constraints and do not consider compiler use cases

---

<!--
私達の焦点はパフォーマンスのためのパフォーマンスではなく、その JS ツールチェインを次のレベルに引き上げる能力を"買う"ためのパフォーマンスです。
-->

## Future of Oxc

Our focus is not on performance for performance's sake, but on performance to "buy" the ability to take that JS toolchain to the next level.

---

# Acknowledgements

- Boshen from Oxc
- All contributors to Oxc, swc, Biome

---

# References

- biomejs/biome (GitHub)
  https://github.com/biomejs/biome
- swc-project/swc (GitHub)
  https://github.com/swc-project/swc
- test262.fyi
  https://test262.fyi/#%7Cchakra,swc
- oxc-project/bench-javascript-parser-written-in-rust (GitHub)
  https://github.com/oxc-project/bench-javascript-parser-written-in-rust
- oxc-project/oxc (GitHub)
  https://github.com/oxc-project/oxc
