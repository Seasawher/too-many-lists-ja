# Basic Data Layout

<!-- Alright, so what's a linked list? Well basically, it's a bunch of pieces of data
on the heap (hush, kernel people!) that point to each other in sequence. Linked
lists are something procedural programmers shouldn't touch with a 10-foot pole,
and what functional programmers use for everything. It seems fair, then, that we
should ask functional programmers for the definition of a linked list. They will
probably give you something like the following definition: -->

さて，連結リストとは何でしょうか．基本的にはヒープ (カーネルのひとたち，静かに！) 上にある，
順番にお互いを指すデータの断片の束のことです．連結リストは手続き型のプログラマにとっては
長い棒でつんつんするのも避けたい代物ですが，関数型のプログラマはあらゆることに使います．
それでは関数型プログラマに連結リストの定義を訊いてみましょう．きっとこんな返事が返ってくると思います:

```haskell
List a = Empty | Elem a (List a)
```

<!-- Which reads approximately as "A List is either Empty or an Element followed by a
List". This is a recursive definition expressed as a *sum type*, which is a
fancy name for "a type that can have different values which may be different
types". Rust calls sum types `enum`s! If you're coming from a C-like language,
this is exactly the enum you know and love, but in overdrive. So let's
transcribe this functional definition into Rust! -->

これは，おおよそ「リストは空か，リストに続く要素のどちらかである」と読めます．これは再帰的な定義を
タグ付き共用型 (sum type, tagged union type) を使って表したものです．ただし，タグ付き共用型というのは
「異なる型の値を持つことができる型」というのをおしゃれに言ったものです．Rust ではこれを `enum` (列挙型)
と呼んでいます．あなたが C 系の言語をすでにご存じなら，話が早いかもしれません．Rust の `enum` は，あなたが
知っている（そして愛している）`enum` と同じものです．では，先ほどの関数型定義を Rust で書き直してみましょう．

<!-- For now we'll avoid generics to keep things simple. We'll only support
storing signed 32-bit integers: -->

今のところ，シンプルにするためジェネリクスは使わないでおきます．符号付き 32 ビット整数 `i32` のみを格納することにします．

```rust ,ignore
// in first.rs

// このモジュールを外部のひとが使えるように pub を付けています
pub enum List {
    Empty,
    Elem(i32, List),
}
```

<!-- *phew*, I'm swamped. Let's just go ahead and compile that: -->

ふー，忙しい忙しい．さっさとコンパイルしちゃいましょう．

```text
> cargo build

error[E0072]: recursive type `first::List` has infinite size
 --> src/first.rs:4:1
  |
4 | pub enum List {
  | ^^^^^^^^^^^^^ recursive type has infinite size
5 |     Empty,
6 |     Elem(i32, List),
  |               ---- recursive without indirection
  |
  = help: insert indirection (e.g., a `Box`, `Rc`, or `&`) at some point to make `first::List` representable
```

<!-- Well. I don't know about you, but I certainly feel betrayed by the functional
programming community. -->

なんだよ，関数型プログラマに騙された！（個人の意見です）

<!-- If we actually check out the error message (after we get over the whole
betrayal thing), we can see that rustc is actually telling us exactly
how to solve this problem: -->

(裏切り行為を乗り越えた後) 実際にエラーメッセージを確認すると，実は rustc がこの問題を解決する方法を
正確に教えてくれていることがわかります．

<!-- > insert indirection (e.g., a `Box`, `Rc`, or `&`) at some point to make `first::List` representable -->

> `first::List` を表すために，どこかで `Box`, `Rc`, `&` などの間接参照を用いてください

<!-- Alright, `box`. What's that? Let's google `rust box`... -->

さてさて，`Box` ってなんでしょうね？ `Rust box` でググってみよう...．

> [std::boxed::Box - Rust](https://doc.rust-lang.org/std/boxed/struct.Box.html)

<!-- Lesse here... -->

どれどれ...．

<!-- > `pub struct Box<T>(_);`
> A pointer type for heap allocation.
> See the [module-level documentation](https://doc.rust-lang.org/std/boxed/) for more. -->
> `pub struct Box<T>(_);`
> 
> ヒープ割り当てのためのポインタ型．詳しくは，[モジュールごとのドキュメント]((https://doc.rust-lang.org/std/boxed/))を参照してください．

<!-- *clicks link* -->

リンクをクリック．

<!-- > `Box<T>`, casually referred to as a 'box', provides the simplest form of heap allocation in Rust. Boxes provide ownership for this allocation, and drop their contents when they go out of scope.
>
> Examples
>
> Creating a box:
>
> `let x = Box::new(5);`
>
> Creating a recursive data structure:
> -->
> `Box<T>` (通称 `box`) は，Rust で最も単純なヒープ割り当ての形式です．ボックスは，この割り当ての所有権を持ち，ボックスがスコープを抜けるとその内容が破棄されます．
> 
> 例
> 
> ボックスを作る:
> 
> `let x = Box::new(5);`
> 
> 再帰的なデータ構造を作る:
```rust
#[derive(Debug)]
enum List<T> {
    Cons(T, Box<List<T>>),
    Nil,
}
```
>
```rust ,ignore
fn main() {
    let list: List<i32> = List::Cons(1, Box::new(List::Cons(2, Box::new(List::Nil))));
    println!("{:?}", list);
}
```
<!-- >
> This will print `Cons(1, Box(Cons(2, Box(Nil))))`.
>
> Recursive structures must be boxed, because if the definition of Cons looked like this:
>
> `Cons(T, List<T>),`
>
> It wouldn't work. This is because the size of a List depends on how many elements are in the list, and so we don't know how much memory to allocate for a Cons. By introducing a Box, which has a defined size, we know how big Cons needs to be. -->
> 上のコードは `Cons(1, Box(Cons(2, Box(Nil))))` と出力します．
> 
> 次のような形の再帰的な定義はボックス化する必要があります．
> 
> `Cons(T, List<T>),`
> 
> そのままでは上手くいきません．なぜなら，リストのサイズがリスト自身に含まれている要素の数に依存するので，
> `Cons` に割り当てるべきメモリの量がわからないからです．ボックスはサイズが決まっているので，`Cons` に
> 必要な大きさを事前に知ることができます．

<!-- Wow, uh. That is perhaps the most relevant and helpful documentation I have ever seen. Literally the first thing in the documentation is *exactly what we're trying to write, why it didn't work, and how to fix it*. -->

ワーオ．こんなに適切で役に立つドキュメントは見たことがありませんね．私たちが書いたものがなぜ上手くいかなかったのか，
そしてどう修正すれば良いのかがずばり書かれています．

<!-- Dang, docs rule. -->

やっぱりドキュメントは最高だな！

<!-- Ok, let's do that: -->

よし，やってみよう．

```rust ,ignore
pub enum List {
    Empty,
    Elem(i32, Box<List>),
}
```

```text
> cargo build

   Finished dev [unoptimized + debuginfo] target(s) in 0.22s
```

<!-- Hey it built! -->

やった，ビルドが通ったぞ！

<!-- ...but this is actually a really foolish definition of a List, for a few reasons. -->

しかし，これはいくつかの理由から，実に愚かな定義です．

<!-- Consider a list with two elements: -->

2つの要素を持つリストを考えてみましょう:

```text
[] = スタック
() = ヒープ

[Elem A, ptr] -> (Elem B, ptr) -> (Empty, *junk*)
```

<!-- There are two key issues: -->

重要な問題が2つあります．

<!-- * We're allocating a node that just says "I'm not actually a Node"
* One of our nodes isn't heap-allocated at all. -->

* 「ぼくはノードじゃない」というだけのノードを割り当てている．
*  ノードの一つに全くヒープが確保されていない．

<!-- On the surface, these two seem to cancel each-other out. We heap-allocate an
extra node, but one of our nodes doesn't need to be heap-allocated at all.
However, consider the following potential layout for our list: -->

表面的には，この2つは互いに打ち消しあうように見えます．余分なノードをヒープに割り当てる一方で，
ノードの１つはヒープではなくスタックに割り当てています．しかし次のような設計のリストを考えてみてください．

```text
[ptr] -> (Elem A, ptr) -> (Elem B, *null*)
```

<!-- In this layout we now unconditionally heap allocate our nodes. The
key difference is the absence of the *junk* from our first layout. What is
this junk? To understand that, we'll need to look at how an enum is laid out
in memory. -->

この設計では，ノードは常にヒープに割り当てています．最初の設計との重要な違いは，ジャンクがないことです．
このジャンクとは何でしょうか？それを理解するために，`enum` がどのようにメモリ上に配置される
かを見る必要があります．

<!-- In general, if we have an enum like: -->

一般的に，次のような `enum` があったとします:

```rust ,ignore
enum Foo {
    D1(T1),
    D2(T2),
    ...
    Dn(Tn),
}
```

<!-- A Foo will need to store some integer to indicate which *variant* of the enum it
represents (`D1`, `D2`, .. `Dn`). This is the *tag* of the enum. It will also
need enough space to store the *largest* of `T1`, `T2`, .. `Tn` (plus some extra
space to satisfy alignment requirements). -->

Foo は，それがどの列挙型のヴァリアント (`D1`, `D2`, .. `Dn`) を表しているかを示す整数を格納する必要があります．
これを列挙型のタグといいます．また，`T1`, `T2`, .. `Tn` のうち最大のものを格納できるだけのスペースも必要です．
(さらに，アラインメントの要件を満たすための余分なスペースも必要)．

<!-- The big takeaway here is that even though `Empty` is a single bit of
information, it necessarily consumes enough space for a pointer and an element,
because it has to be ready to become an `Elem` at any time. Therefore the first
layout heap allocates an extra element that's just full of junk, consuming a
bit more space than the second layout. -->

ここで重要なのは，`Empty` が1ビットの情報であっても，いつでも `Elem` になれるように準備しなければならないので，
ポインタと要素のために必要なスペースを必ず消費する，ということです．したがって，最初の設計ではヒープにジャンクでいっぱいの
余分な要素が割り当てられ，2つめの設計よりも少し多くのスペースを消費してしまうのです．

<!-- One of our nodes not being allocated at all is also, perhaps surprisingly,
*worse* than always allocating it. This is because it gives us a *non-uniform*
node layout. This doesn't have much of an appreciable effect on pushing and
popping nodes, but it does have an effect on splitting and merging lists. -->

ノードの一つをヒープに割り当てていないことも，悪いことです．常にヒープに割り当てる方がましかもしれません．これはノードの設計が不均一になるからです．
ノードのプッシュやポップにはあまり影響しませんが，リストの分割やマージには影響します．

<!-- Consider splitting a list in both layouts: -->

リストの分割を行ったときにどうなるか，両方の設計を比較してみましょう:

<!-- ```text
layout 1:

[Elem A, ptr] -> (Elem B, ptr) -> (Elem C, ptr) -> (Empty *junk*)

split off C:

[Elem A, ptr] -> (Elem B, ptr) -> (Empty *junk*)
[Elem C, ptr] -> (Empty *junk*)
``` -->

```text
設計 1:

[Elem A, ptr] -> (Elem B, ptr) -> (Elem C, ptr) -> (Empty *junk*)

C で分割すると:

[Elem A, ptr] -> (Elem B, ptr) -> (Empty *junk*)
[Elem C, ptr] -> (Empty *junk*)
```

<!-- ```text
layout 2:

[ptr] -> (Elem A, ptr) -> (Elem B, ptr) -> (Elem C, *null*)

split off C:

[ptr] -> (Elem A, ptr) -> (Elem B, *null*)
[ptr] -> (Elem C, *null*)
``` -->

```text
設計 2:

[ptr] -> (Elem A, ptr) -> (Elem B, ptr) -> (Elem C, *null*)

C で分割すると:

[ptr] -> (Elem A, ptr) -> (Elem B, *null*)
[ptr] -> (Elem C, *null*)
```

<!-- Layout 2's split involves just copying B's pointer to the stack and nulling
the old value out. Layout 1 ultimately does the same thing, but also has to
copy C from the heap to the stack. Merging is the same process in reverse. -->

設計2の分割では，Bのポインタをスタックにコピーし，古いポインタを `null` にするだけです．
設計1では同じことをするために，さらにCをヒープからスタックにコピーする必要があります．
逆になるだけで，マージの場合も同様です．

<!-- One of the few nice things about a linked list is that you can construct the
element in the node itself, and then freely shuffle it around lists without
ever moving it. You just fiddle with pointers and stuff gets "moved". Layout 1
trashes this property. -->

ノード自体に要素を構築し，それを動かすことなくリスト間で自由に並び替えることが
できる点が，連結リストの数少ない長所でした．ポインターをいじるだけで，ものが「移動」するのです．
設計1ではこの性質がゴミと化しています．

Alright, I'm reasonably convinced Layout 1 is bad. How do we rewrite our List?
Well, we could do something like:

```rust ,ignore
pub enum List {
    Empty,
    ElemThenEmpty(i32),
    ElemThenNotEmpty(i32, Box<List>),
}
```

Hopefully this seems like an even worse idea to you. Most notably, this really
complicates our logic, because there is now a completely invalid state:
`ElemThenNotEmpty(0, Box(Empty))`. It also *still* suffers from non-uniformly
allocating our elements.

However it does have *one* interesting property: it totally avoids allocating
the Empty case, reducing the total number of heap allocations by 1. Unfortunately,
in doing so it manages to waste *even more space*! This is because the previous
layout took advantage of the *null pointer optimization*.

We previously saw that every enum has to store a *tag* to specify which variant
of the enum its bits represent. However, if we have a special kind of enum:

```rust,ignore
enum Foo {
    A,
    B(ContainsANonNullPtr),
}
```

the null pointer optimization kicks in, which *eliminates the space needed for
the tag*. If the variant is A, the whole enum is set to all `0`'s. Otherwise,
the variant is B. This works because B can never be all `0`'s, since it contains
a non-zero pointer. Slick!

Can you think of other enums and types that could do this kind of optimization?
There's actually a lot! This is why Rust leaves enum layout totally unspecified.
There are a few more complicated enum layout optimizations that Rust will do for
us, but the null pointer one is definitely the most important!
It means `&`, `&mut`, `Box`, `Rc`, `Arc`, `Vec`, and
several other important types in Rust have no overhead when put in an `Option`!
(We'll get to most of these in due time.)

So how do we avoid the extra junk, uniformly allocate, *and* get that sweet
null-pointer optimization? We need to better separate out the idea of having an
element from allocating another list. To do this, we have to think a little more
C-like: structs!

While enums let us declare a type that can contain *one* of several values,
structs let us declare a type that contains *many* values at once. Let's break
our List into two types: A List, and a Node.

As before, a List is either Empty or has an element followed by another List.
By representing the "has an element followed by another List" case by an
entirely separate type, we can hoist the Box to be in a more optimal position:

```rust ,ignore
struct Node {
    elem: i32,
    next: List,
}

pub enum List {
    Empty,
    More(Box<Node>),
}
```

Let's check our priorities:

* Tail of a list never allocates extra junk: check!
* `enum` is in delicious null-pointer-optimized form: check!
* All elements are uniformly allocated: check!

Alright! We actually just constructed exactly the layout that we used to
demonstrate that our first layout (as suggested by the official Rust
documentation) was problematic.

```text
> cargo build

warning: private type `first::Node` in public interface (error E0446)
 --> src/first.rs:8:10
  |
8 |     More(Box<Node>),
  |          ^^^^^^^^^
  |
  = note: #[warn(private_in_public)] on by default
  = warning: this was previously accepted by the compiler but
    is being phased out; it will become a hard error in a future release!
```

:(

Rust is mad at us again. We marked the `List` as public (because we want people
to be able to use it), but not the `Node`. The problem is that the internals of
an `enum` are totally public, and we're not allowed to publicly talk about
private types. We could make all of `Node` totally public, but generally in Rust
we favour keeping implementation details private. Let's make `List` a struct, so
that we can hide the implementation details:

```rust ,ignore
pub struct List {
    head: Link,
}

enum Link {
    Empty,
    More(Box<Node>),
}

struct Node {
    elem: i32,
    next: Link,
}
```

Because `List` is a struct with a single field, its size is the same as that
field. Yay zero-cost abstractions!

```text
> cargo build

warning: field is never used: `head`
 --> src/first.rs:2:5
  |
2 |     head: Link,
  |     ^^^^^^^^^^
  |
  = note: #[warn(dead_code)] on by default

warning: variant is never constructed: `Empty`
 --> src/first.rs:6:5
  |
6 |     Empty,
  |     ^^^^^

warning: variant is never constructed: `More`
 --> src/first.rs:7:5
  |
7 |     More(Box<Node>),
  |     ^^^^^^^^^^^^^^^

warning: field is never used: `elem`
  --> src/first.rs:11:5
   |
11 |     elem: i32,
   |     ^^^^^^^^^

warning: field is never used: `next`
  --> src/first.rs:12:5
   |
12 |     next: Link,
   |     ^^^^^^^^^^

```

Alright, that compiled! Rust is pretty mad, because as far as it can tell,
everything we've written is totally useless: we never use `head`, and no one who
uses our library can either since it's private. Transitively, that means Link
and Node are useless too. So let's solve that! Let's implement some code for our
List!
