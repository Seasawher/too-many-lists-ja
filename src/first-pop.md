<!-- # Pop -->
# Pop メソッド

<!-- Like `push`, `pop` wants to mutate the list. Unlike `push`, we actually
want to return something. But `pop` also has to deal with a tricky corner
case: what if the list is empty? To represent this case, we use the trusty
`Option` type: -->

`push` と同様，`pop` はリストを変更するメソッドです．
`push` と異なるところは，実際に何かしらの値を返すことですね．
`pop` はリストが空だったらどうするかという厄介なコーナーケースも扱う必要があります．
このケースに対処するために，`Option` 型という頼もしい道具を使用します:

```rust ,ignore
pub fn pop(&mut self) -> Option<i32> {
    // TODO
}
```

<!-- `Option<T>` is an enum that represents a value that may exist. It can either be
`Some(T)` or `None`. We could make our own enum for this like we did for
Link, but we want our users to be able to understand what the heck our return
type is, and Option is so ubiquitous that *everyone* knows it. In fact, it's so
fundamental that it's implicitly imported into scope in every file, as well
as its variants `Some` and `None` (so we don't have to say `Option::None`). -->

`Option<T>` は存在するかどうかわからない値を扱う列挙型です．
`Some(T)` か `None` かのいずれかの値を取ります．
`Link` のときにやったように独自の `enum` を作ってもいいのですが，ユーザに戻り値が何なのか理解してほしいので，
どこにでもあり **誰もが** 知っている `Option` 型を使うのが得策です．
実際，`Option` は非常に基本的なものなので，すべてのファイルで暗黙のうちにスコープに取り込まれます．
そのヴァリアントである `Some` や `None` も同様です．
(ですから，`Option::None` などと書く必要はないのです)

<!-- The pointy bits on `Option<T>` indicate that Option is actually *generic* over
T. That means that you can make an Option for *any* type! -->

`Option<T>` の `<T>` は，`Option` が `T` に対するジェネリックであることを示しています．
つまり，どんな型に対しても `Option` を作ることができるのです！

<!-- So uh, we have this `Link` thing, how do we figure out if it's Empty or has
More? Pattern matching with `match`! -->

さて，この `Link` ですが，`Empty` か `More` のどちらなのか，どうやって判断すればいいでしょう?
`match` でパターンマッチしてみますか！

```rust ,ignore
pub fn pop(&mut self) -> Option<i32> {
    match self.head {
        Link::Empty => {
            // TODO
        }
        Link::More(node) => {
            // TODO
        }
    };
}
```

```text
> cargo build

error[E0308]: mismatched types
  --> src/first.rs:27:30
   |
27 |     pub fn pop(&mut self) -> Option<i32> {
   |            ---               ^^^^^^^^^^^ expected enum `std::option::Option`, found ()
   |            |
   |            this function's body doesn't return
   |
   = note: expected type `std::option::Option<i32>`
              found type `()`
```

<!-- Whoops, `pop` has to return a value, and we're not doing that yet. We *could*
return `None`, but in this case it's probably a better idea to return
`unimplemented!()`, to indicate that we aren't done implementing the function.
`unimplemented!()` is a macro (`!` indicates a macro) that panics the program
when we get to it (\~crashes it in a controlled manner). -->

おっと，`pop` は値を返さないといけないのに，まだやっていませんでしたね．
`None` を返すこともできますが，この場合，
関数の実装が終わっていないことを示すために `unimplemented!()` を返す方が良いでしょう．
`unimplemented!()` はマクロで (`!` はマクロを表します)，これに到達するとプログラムがパニックします．
(パニックとは，制御された方法でプログラムをクラッシュさせること)．

```rust ,ignore
pub fn pop(&mut self) -> Option<i32> {
    match self.head {
        Link::Empty => {
            // TODO
        }
        Link::More(node) => {
            // TODO
        }
    };
    unimplemented!()
}
```

<!-- Unconditional panics are an example of a [diverging function][diverging].
Diverging functions never return to the caller, so they may be used in places
where a value of any type is expected. Here, `unimplemented!()` is being
used in place of a value of type `Option<T>`. -->

無条件のパニックは，発散する関数 ([diverging function][diverging]) の例です．
発散する関数は呼び出し元に戻ることがないので，本来期待される値の型がなんであったとしても，
使用することができます．
ここでは，`Option<T>` 型の代わりに `unimplemented!()` を使用しています．

<!-- Note also that we don't need to write `return` in our program. The last
expression (basically line) in a function is implicitly its return value. This
lets us express really simple things a bit more concisely. You can always
explicitly return early with `return` like any other C-like language. -->

また，プログラム中に `return` を書く必要はないことに注意してください．
関数の最後の式 (基本的には行) が，暗黙のうちに戻り値になります．
これによって，すごく単純な関数をさらに単純にすることができます．
他の C 系の言語と同様に，`return` で早めに値を明示的に返すこともできます．

```text
> cargo build

error[E0507]: cannot move out of borrowed content
  --> src/first.rs:28:15
   |
28 |         match self.head {
   |               ^^^^^^^^^
   |               |
   |               cannot move out of borrowed content
   |               help: consider borrowing here: `&self.head`
...
32 |             Link::More(node) => {
   |                        ---- data moved here
   |
note: move occurs because `node` has type `std::boxed::Box<first::Node>`, which does not implement the `Copy` trait
  --> src/first.rs:32:24
   |
32 |             Link::More(node) => {
   |                        ^^^^
```

<!-- Come on Rust, get off our back! As always, Rust is hella mad at us. Thankfully,
this time it's also giving us the full scoop! By default, a pattern match will
try to move its contents into the new branch, but we can't do this because we
don't own self by-value here. -->

おい Rust, 俺たちの邪魔をするな！
いつものように，Rust がカンカンに起こっています．
でも今回はちゃんと理由を教えてくれました．
デフォルトでは，パターンマッチは中身を新しいブランチにムーブしようとしますが，
ここでは `self` を値として所有していないため，これが行えないのです．

<!-- ```text
help: consider borrowing here: `&self.head`
``` -->

```text
help: consider borrowing here: `&self.head`

ヘルプ: 借用することを検討してみてください: `&self.head`
```

<!-- Rust says we should add a reference to our `match` to fix that. 🤷‍♀️ Let's try it: -->

Rust が言うには，`match` を参照にすれば直るみたいですね．🤷‍♀️
試してみましょう:

```rust ,ignore
pub fn pop(&mut self) -> Option<i32> {
    match &self.head {
        Link::Empty => {
            // TODO
        }
        Link::More(node) => {
            // TODO
        }
    };
    unimplemented!()
}
```

```text
> cargo build

warning: unused variable: `node`
  --> src/first.rs:32:24
   |
32 |             Link::More(node) => {
   |                        ^^^^ help: consider prefixing with an underscore: `_node`
   |
   = note: #[warn(unused_variables)] on by default

warning: field is never used: `elem`
  --> src/first.rs:13:5
   |
13 |     elem: i32,
   |     ^^^^^^^^^
   |
   = note: #[warn(dead_code)] on by default

warning: field is never used: `next`
  --> src/first.rs:14:5
   |
14 |     next: Link,
   |     ^^^^^^^^^^
```

Hooray, compiling again! Now let's figure out that logic. We want to make an
Option, so let's make a variable for that. In the Empty case we need to return
None. In the More case we need to return `Some(i32)`, and change the head of
the list. So, let's try to do basically that?

```rust ,ignore
pub fn pop(&mut self) -> Option<i32> {
    let result;
    match &self.head {
        Link::Empty => {
            result = None;
        }
        Link::More(node) => {
            result = Some(node.elem);
            self.head = node.next;
        }
    };
    result
}
```

```text
> cargo build
   Compiling lists v0.1.0 (/Users/ABeingessner/dev/temp/lists)
error[E0507]: cannot move out of borrowed content
  --> src/first.rs:35:29
   |
35 |                 self.head = node.next;
   |                             ^^^^^^^^^ cannot move out of borrowed content

```

*head*

*desk*

We're trying to move out of `node` when all we have is a shared reference to it.

We should probably step back and think about what we're trying to do. We want
to:

* Check if the list is empty.
* If it's empty, just return None
* If it's *not* empty
    * remove the head of the list
    * remove its `elem`
    * replace the list's head with its `next`
    * return `Some(elem)`

The key insight is we want to *remove* things, which means we want to get the
head of the list *by value*. We certainly can't do that through the shared
reference we get through `&self.head`. We also "only" have a mutable reference
to `self`, so the only way we can move stuff is to *replace it*. Looks like we're doing
the Empty dance again!

Let's try that:


```rust ,ignore
pub fn pop(&mut self) -> Option<i32> {
    let result;
    match mem::replace(&mut self.head, Link::Empty) {
        Link::Empty => {
            result = None;
        }
        Link::More(node) => {
            result = Some(node.elem);
            self.head = node.next;
        }
    };
    result
}
```

```text
cargo build

   Finished dev [unoptimized + debuginfo] target(s) in 0.22s
```

O M G

It compiled without *any* warnings!!!!!

Actually I'm going to apply my own personal lint here: we made this `result`
value to return, but actually we didn't need to do that at all! Just as a
function evaluates to its last expression, every block also evaluates to
its last expression. Normally we supress this behaviour with semi-colons,
which instead makes the block evaluate to the empty tuple, `()`. This is
actually the value that functions which don't declare a return value -- like
`push` -- return.

So instead, we can write `pop` as:

```rust ,ignore
pub fn pop(&mut self) -> Option<i32> {
    match mem::replace(&mut self.head, Link::Empty) {
        Link::Empty => None,
        Link::More(node) => {
            self.head = node.next;
            Some(node.elem)
        }
    }
}
```

Which is a bit more concise and idiomatic. Note that the Link::Empty branch
completely lost its braces, because we only have one expression to
evaluate. Just a nice shorthand for simple cases.

```text
cargo build

   Finished dev [unoptimized + debuginfo] target(s) in 0.22s
```

Nice, still works!



[ownership]: first-ownership.html
[diverging]: https://doc.rust-lang.org/nightly/book/ch19-04-advanced-types.html#the-never-type-that-never-returns
