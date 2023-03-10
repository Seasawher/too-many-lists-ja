<!-- # Push -->
# Push メソッド

<!-- So let's write pushing a value onto a list. `push` *mutates* the list,
so we'll want to take `&mut self`. We also need to take an i32 to push: -->

リストに値をプッシュする操作を実装していきましょう．
`push` はリストを変更するので，`&mut self` を引数に取ります．
さらに，プッシュする値が必要なので `i32` 型の変数も引数に用意します．

```rust ,ignore
impl List {
    pub fn push(&mut self, elem: i32) {
        // TODO
    }
}
```

<!-- First things first, we need to make a node to store our element in: -->

まず最初に，要素を格納するためのノードを作ります:

```rust ,ignore
    pub fn push(&mut self, elem: i32) {
        let new_node = Node {
            elem: elem,
            next: ?????
        };
    }
```

<!-- What goes `next`? Well, the entire old list! Can we... just do that? -->

`next` の中身はなんでしょう? 以前のリスト全体ですね！
これで…できるかな？

```rust ,ignore
impl List {
    pub fn push(&mut self, elem: i32) {
        let new_node = Node {
            elem: elem,
            next: self.head,
        };
    }
}
```

```text
> cargo build
error[E0507]: cannot move out of borrowed content
  --> src/first.rs:19:19
   |
19 |             next: self.head,
   |                   ^^^^^^^^^ cannot move out of borrowed content
```

<!-- Nooooope. Rust is telling us the right thing, but it's certainly not obvious
what exactly it means, or what to do about it: -->

あーもう！Rust は正しいことを言ってるんでしょうが，具体的に何を意味しているのか，
どうすればいいのかわかりません:

> cannot move out of borrowed content
> 
> 借用中の値をムーブすることはできません

<!-- We're trying to move the `self.head` field out to `next`, but Rust doesn't want
us doing that. This would leave `self` only partially initialized when we end
the borrow and "give it back" to its rightful owner. As we said before, that's
the *one* thing you can't do with an `&mut`: It would be super rude,
and Rust is very polite (it would also be incredibly dangerous, but surely
*that* isn't why it cares). -->

私たちは今 `self.head` フィールドを `next` にムーブしようとしましたが，それが Rust のお咎めを受けました．
これでは，借用を終了して正当な所有者に「返す」ときに，部分的にしか初期化されていない `self` を返すことになります．
先に述べたように，`&mut` でこんなことはできません．
Rust は非常に礼儀正しいからです．こんな失礼なことはあってはなりません．
(びっくりするほど危険なことでもありますが，礼を失することに比べればとるに足りないことです)．

<!-- What if we put something back? Namely, the node that we're creating: -->

ムーブされた部分を埋めてみたらどうでしょうか？つまり，いま作ったリストを返すわけです:


```rust ,ignore
pub fn push(&mut self, elem: i32) {
    let new_node = Box::new(Node {
        elem: elem,
        next: self.head,
    });

    self.head = Link::More(new_node);
}
```

```text
> cargo build
error[E0507]: cannot move out of borrowed content
  --> src/first.rs:19:19
   |
19 |             next: self.head,
   |                   ^^^^^^^^^ cannot move out of borrowed content
```

<!-- No dice. In principle, this is something Rust could actually accept, but it
won't (for various reasons -- the most serious being [exception safety][]). We need
some way to get the head without Rust noticing that it's gone. For advice, we
turn to infamous Rust Hacker Indiana Jones: -->

ダメでした．実際には Rust はこれを受け入れてもよいのですが，（様々な理由 -- 最も深刻なのは[例外安全性][]です -- により）
受け容れません．
Rust に気づかれないように，`head` を取得する方法が必要なのです．
そこで，悪名高き Rust ハッカー，インディアナ・ジョーンズに助言を求めましょう:

![Indy Prepares to mem::replace](img/indy.gif)

<!-- Ah yes, Indy suggests the `mem::replace` maneuver. This incredibly useful
function lets us steal a value out of a borrow by *replacing* it with another
value. Let's just pull in `std::mem` at the top of the file, so that `mem` is in
local scope: -->

おっ，インディは `mem::replace` を使うことを提案していますね．
この信じられないほど便利な関数を使えば，値を別の値で **置き換える** ことで，借りた値を盗むことができます．
`mem` を使用するためには，ファイルの先頭に `std::mem` を引っ張ってきたうえで:

```rust ,ignore
use std::mem;
```

<!-- and use it appropriately: -->
使うべきところで使えばよいです:

```rust ,ignore
pub fn push(&mut self, elem: i32) {
    let new_node = Box::new(Node {
        elem: elem,
        next: mem::replace(&mut self.head, Link::Empty),
    });

    self.head = Link::More(new_node);
}
```

<!-- Here we `replace` self.head temporarily with Link::Empty before replacing it
with the new head of the list. I'm not gonna lie: this is a pretty unfortunate
thing to have to do. Sadly, we must (for now). -->

ここでは `self.head` を一時的に `Link::Empty` に `replace` してから，リストの新しい `head` に置き換えています．
率直に言って，これはかなり不満の残るコードです．しかし悲しいかな，そうせざるを得ないのですよ．
今のところはね．

<!-- But hey, that's `push` all done! Probably. We should probably test it, honestly.
Right now the easiest way to do that is probably to write `pop`, and make sure
that it produces the right results. -->

でも，これで `push` メソッドが完成しました！たぶんだけど．
正直，テストをしないとわかりません．
今テストをする最も簡単な方法は，おそらく `pop` も書いて，正しい結果が得られるか確認することでしょう．

[例外安全性]: https://doc.rust-lang.org/nightly/nomicon/exception-safety.html
