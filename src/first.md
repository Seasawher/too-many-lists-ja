<!-- # A Bad Singly-Linked Stack -->
# 単方向リストのだめな実装

<!-- This one's gonna be *by far* the longest, as we need to introduce basically
all of Rust, and are gonna build up some things "the hard way" to better
understand the language. -->

Rust 言語の基本的な部分をすべて紹介し，よく理解するために「難しい方法」で作っていくという方針をとったため，この章はとても長くなりました．

<!-- We'll put our first list in `src/first.rs`. We need to tell Rust that `first.rs` is
something that our lib uses. All that requires is that we put this at the top of
`src/lib.rs` (which Cargo made for us): -->

最初のリストは `src/first.rs` に置きます．Rust に `first.rs` が私たちのライブラリを使用するファイルだということを知らせる必要がありますが，
そのためには (Cargo が作ってくれた) `src/lib.rs` の最初の行に次のように記述します．

```rust ,ignore
// in lib.rs
pub mod first;
```
