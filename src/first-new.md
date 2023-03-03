<!-- # New -->
# New メソッド

<!-- To associate actual code with a type, we use `impl` blocks: -->

実際のコードを型に関連付けるには，`impl` ブロックを使用します:

<!-- ```rust ,ignore
impl List {
    // TODO, make code happen
}
``` -->

```rust ,ignore
impl List {
    // TODO, コードを書く
}
```

<!-- Now we just need to figure out how to actually write code. In Rust we declare
a function like so: -->

後は，実際にどんなコードを書くか考えるだけです．Rust では，次のように関数を宣言します．

<!-- ```rust ,ignore
fn foo(arg1: Type1, arg2: Type2) -> ReturnType {
    // body
}
``` -->

```rust ,ignore
fn foo(arg1: Type1, arg2: Type2) -> ReturnType {
    // 関数の中で行う処理
}
```

<!-- The first thing we want is a way to *construct* a list. Since we hide the
implementation details, we need to provide that as a function. The usual way
to do that in Rust is to provide a static method, which is just a
normal function inside an `impl`: -->

最初に欲しいのは，リストを新しく作る方法です．実装の詳細は隠しておきたいので，
それを関数として提供する必要があります．Rust では，こういう場合には静的メソッドを使用するのが一般的です．
静的メソッドは，`impl` ブロックの中にある普通の関数です:

```rust ,ignore
impl List {
    pub fn new() -> Self {
        List { head: Link::Empty }
    }
}
```

<!-- A few notes on this: -->

ここで注釈を少し:

<!-- * Self is an alias for "that type I wrote at the top next to `impl`". Great for
  not repeating yourself!
* We create an instance of a struct in much the same way we declare it, except
  instead of providing the types of its fields, we initialize them with values.
* We refer to variants of an enum using `::`, which is the namespacing operator.
* The last expression of a function is implicitly returned.
  This makes simple functions a little neater. You can still use `return`
  to return early like other C-like languages. -->

* `Self` は「`impl` の横の，いちばん上に書いたあの型」の別名です．これで同じコードを繰り返さずに済みます．
* 構造体のインスタンスは，構造体を宣言するのとほぼ同じ方法で作成できますが，フィールドの型を指定する代わりに，値を与えて初期化します．
* `enum` のヴァリアントは，名前空間演算子である `::` を使って参照します．
* 関数は暗黙のうちに最後の式を返します．これにより単純な関数が少しすっきりします．他の C 系の言語と同様に，`return` を使って関数の終端に来る前に値を返すことも可能です．





















