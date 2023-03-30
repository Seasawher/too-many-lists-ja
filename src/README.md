<!-- # Learn Rust With Entirely Too Many Linked Lists -->
# 連結リストまみれでRustを学ぶ

<!-- > Got any issues or want to check out all the final code at once?
> [Everything's on Github!][github] -->

> 何か問題を見つけましたか？または，最終的なコードを一気にチェックしたいですか？すべて[GitHub][github]にあります！

> `訳注`: 日本語版のリポジトリは[こちら](https://github.com/Seasawher/too-many-lists-ja)です

<!-- > **NOTE**: The current edition of this book is written against Rust 2018,
> which was first released with rustc 1.31 (Dec 8, 2018). If your rust toolchain
> is new enough, the Cargo.toml file that `cargo new` creates should contain the
> line `edition = "2018"` (or if you're reading this in the far future, perhaps
> some even larger number!). Using an older toolchain is possible, but unlocks
> a secret **hardmode**, where you get extra compiler errors that go completely
> unmentioned in the text of this book. Wow, sounds like fun! -->

> **注意**: 本書の現行版は，rustc 1.31 (2018年12月8日) に初めてリリースされた Rust 2018 に基づいて書かれています．
> あなたの Rust ツールチェインが十分に新しい場合，`cargo new` が生成する `Cargo.toml` ファイルには `edition = "2018"` 
> (あるいは，あなたが遠い未来に読んでいるなら，もっと大きな数字！) という行が含まれているはずです．
> 古いツールチェインを利用することも可能ですが，それをすると秘密のハードモードが解除され，
> 本書の本文ではまったく触れられないコンパイルエラーが発生します．ウワー，楽しそう！

<!-- I fairly frequently get asked how to implement a linked list in Rust. The
answer honestly depends on what your requirements are, and it's obviously not
super easy to answer the question on the spot. As such I've decided to write
this book to comprehensively answer the question once and for all. -->

Rustで連結リストを実装するにはどうしたらいいか，という質問をよく受けます．正直なところ，答えはお客様のご要望によりますし，その場ですらすらお答えできるものではありません．そこで，そうした疑問をまとめて解決するためにこの本を書くことにしました．

<!-- In this series I will teach you basic and advanced Rust programming
entirely by having you implement 6 linked lists. In doing so, you should
learn: -->

これから６つの連結リストの実装を通して，Rust でのプログラミングの基礎と応用をまるごとお教えします．あなたが学んでいく内容は以下の通り：

<!-- * The following pointer types: `&`, `&mut`, `Box`, `Rc`, `Arc`, `*const`, `*mut`, `NonNull`(?)
* Ownership, borrowing, inherited mutability, interior mutability, Copy
* All The Keywords: struct, enum, fn, pub, impl, use, ...
* Pattern matching, generics, destructors
* Testing, installing new toolchains, using `miri`
* Unsafe Rust: raw pointers, aliasing, stacked borrows, UnsafeCell, variance -->

* 次のポインタ型: `&`, `&mut`, `Box`, `Rc`, `Arc`, `*const`, `*mut`, `NonNull`(?)
* 所有権，借用，継承可変性, 内部可変性，コピー
* 次のキーワードすべて: 構造体，列挙型，`fn`, `pub`, `impl`, `use`,...
* パターンマッチング，ジェネリクス，デストラクタ
* テスト，新しいツールチェインのインストール，`miri` の使用
* アンセーフなRust: 生ポインタ，エイリアス，借用のスタック, UnsafeCell, バリアンス

<!-- Yes, linked lists are so truly awful that you deal with all of these concepts in
making them real. -->

そう，連結リストというのは本当にやばいものなので，実装しようとするとこういった概念を全部扱わないといけないのです．

<!-- Everything's in the sidebar (may be collapsed on mobile), but for quick
reference, here's what we're going to be making: -->

すべてサイドバーにありますが（モバイルなら折りたたまれているかもしれませんけど）ご参考までに簡単にこれから作るものを紹介します．

1. [A Bad Singly-Linked Stack](first.md)
2. [An Ok Singly-Linked Stack](second.md)
3. [A Persistent Singly-Linked Stack](third.md)
4. [A Bad But Safe Doubly-Linked Deque](fourth.md)
5. [An Unsafe Singly-Linked Queue](fifth.md)
6. [TODO: An Ok Unsafe Doubly-Linked Deque](sixth.md)
7. [Bonus: A Bunch of Silly Lists](infinity.md)

<!-- Just so we're all the same page, I'll be writing out all the commands that I
feed into my terminal. I'll also be using Rust's standard package manager, Cargo,
to develop the project. Cargo isn't necessary to write a Rust program, but it's
*so much* better than using rustc directly. If you just want to futz around you
can also run some simple programs in the browser via [play.rust-lang.org][play]. -->

同じ結果を得られるように，私はこれからターミナルで打つコマンドはすべて書き出すつもりです．また開発には Rust の標準パッケージマネージャである Cargo を使うことにします．Cargo を使わなくても Rust プログラムは書けますが，rustc を直接使うよりもずっといいです．ただいじくり回したいだけなら，[Rust プレイグラウンド](https://play.rust-lang.org/)を使ってブラウザ上で実行することもできます．

<!-- In later sections, we'll be using "rustup" to install extra Rust tooling.
I strongly recommend [installing all of your Rust toolchains using rustup](https://www.rust-lang.org/tools/install). -->

この後の章では，`rustup` を使って追加の Rust ツールをインストールしていきます．Rust のツールチェインは[すべてrustup を用いてインストールする](https://www.rust-lang.org/tools/install)ことを強くお勧めします．

<!-- Let's get started and make our project: -->

さっそくプロジェクトを作っていきましょう:

```sh
> cargo new --lib lists
> cd lists
```

<!-- We'll put each list in a separate file so that we don't lose any of our work. -->

各リストを別ファイルにまとめ，作業内容が失われないようにします．

<!-- It should be noted that the *authentic* Rust learning experience involves
writing code, having the compiler scream at you, and trying to figure out
what the heck that means. I will be carefully ensuring that this occurs as
frequently as possible. Learning to read and understand Rust's generally
excellent compiler errors and documentation is *incredibly* important to
being a productive Rust programmer. -->

次のことは言っておかねばならないでしょう: Rust を**本当に**学ぶつもりなら，コードを書き，コンパイラに怒られ，それが何を意味するかを理解するということは避けて通れません．皆さんにお見せするために，ふんだんにエラーを出すようにするつもりです．つよい Rust プログラマになるためには，Rust の優れたコンパイラエラーやドキュメントを読み，理解できるようになることが**とても**重要なのです．

<!-- Although actually that's a lie. In writing this I encountered *way* more
compiler errors than I show. In particular, in the later chapters I won't be
showing a lot of the random "I typed (copy-pasted) bad" errors that you
expect to encounter in every language. This is a *guided tour* of having the
compiler scream at us. -->

ごめんなさい嘘をつきました．私が実際に遭遇したコンパイルエラーの数はこんなものではなく，ここでお見せするのは氷山の一角です．どの言語でも遭遇するようなコピー&ペーストのエラーなどは紹介していません．これはコンパイラに怒られるためのツアーなのです．

<!-- We're going to be going pretty slow, and I'm honestly not going to be very
serious pretty much the entire time. I think programming should be fun, dang it!
If you're the type of person who wants maximally information-dense, serious, and
formal content, this book is not for you. Nothing I will ever make is for you.
You are wrong. -->

かなりスローペースで進めていくので，正直に言ってまじめな目的にはそぐわないでしょう．プログラミングは楽しくなきゃダメだと思うんです，いやほんとに．もしあなたがきまじめなタイプなら，情報がぱんぱんに詰まったお堅くてきっちりしたものを書いてほしいと思われるかもしれませんが，そんな人のために書いているわけではないんです．あなたは間違ってる．

# 注意事項

<!-- Just so we're totally 100% clear: I hate linked lists. With
a passion. Linked lists are terrible data structures. Now of course there's
several great use cases for a linked list: -->

はっきり言っておきますが，私は連結リストが大嫌いです．吐き気を催すほど．連結リストはひどいデータ構造です．もちろん連結リストには以下のようなときには良い選択です:

<!-- * You want to do *a lot* of splitting or merging of big lists. *A lot*.
* You're doing some awesome lock-free concurrent thing.
* You're writing a kernel/embedded thing and want to use an intrusive list.
* You're using a pure functional language and the limited semantics and absence
  of mutation makes linked lists easier to work with.
* ... and more! -->

* 大きなリストの分割や結合をたくさん，たくさん，たくさん！行いたいとき
* 同時並行で複数スレッドがロックせずに進行するようなものを書いているとき
* カーネルまたは埋め込みの開発を行っていて，intrusive list を使いたいとき
* 純粋な関数型言語を使っているとき．この場合はセマンティクスが限定されているしデータが不変なので，連結リストの扱いが簡単

<!-- But all of these cases are *super rare* for anyone writing a Rust program. 99%
of the time you should just use a Vec (array stack), and 99% of the other 1%
of the time you should be using a VecDeque (array deque). These are blatantly
superior data structures for most workloads due to less frequent allocation,
lower memory overhead, true random access, and cache locality. -->

しかし Rust プログラマにとってはこういったことはすべて超レアなケースです．99% のケースでは `Vec` (可変長配列，スタック) を使えば済みますし，残りの 1% のケースも `VecDeque` (両端キュー) で十分です．`Vec` や `VecDeque` は割り当ての頻度が少なく，メモリのオーバーヘッドも小さく，真のランダムアクセスとキャッシュの局所性を備えており，ほとんどの場合に適切なデータ構造です．

<!-- Linked lists are as *niche* and *vague* of a data structure as a trie. Few would
balk at me claiming a trie is a niche structure that your average programmer
could happily never learn in an entire productive career -- and yet linked lists
have some bizarre celebrity status. We teach every undergrad how to write a
linked list. It's the only niche collection
[I couldn't kill from std::collections][rust-std-list]. It's
[*the* list in C++][cpp-std-list]! -->

連結リストはトライ木と同じくらいニッチで，頼りないデータ構造です．トライ木はめったに使用されない構造で，平均的なプログラマが開発の中で使うことはないだろうと主張しても反論はほとんどないと思います．しかし連結リストは奇妙にも名前がよく知られています．学部生の全員が連結リストの書き方を教わりますものね．`std::collections` の中で私が唯一殺すことができなかったニッチなコレクションなんです．C++ のリストなんですよ！

<!-- We should all as a community say *no* to linked lists as a "standard" data
structure. It's a fine data structure with several great use cases, but those
use cases are *exceptional*, not common. -->

私たちはコミュニティとして，連結リストを「標準的な」データ構造として認めるべきではないと思います．連結リストはいくつかの場合には素晴らしいデータ構造ですが，それは例外的なケースあって，一般的に有用というわけではありません．

<!-- Several people apparently read the first paragraph of this PSA and then stop
reading. Like, literally they'll try to rebut my argument by listing one of the
things in my list of *great use cases*. The thing right after the first
paragraph! -->

この節の最初の段落を読んだだけで，この本を読むのをやめてしまうひとがいます．例えば，私が2段落目で示した連結リストの素晴らしい使用例の一つを挙げて，反論してこようとするのです．

<!-- Just so I can link directly to a detailed argument, here are several attempts
at counter-arguments I have seen, and my response to them. Feel free to skip
to [the first chapter](first.md) if you just want to learn some Rust! -->

私に対するいくつかの反論と，それに対する私からの再反論の詳細を以下に示しておきますが，Rust を学びたい方は[第1章](first.md)まで読み飛ばしてください．


<!-- ## Performance doesn't always matter -->
## パフォーマンスは必ずしも問題ではない

<!-- Yes! Maybe your application is I/O-bound or the code in question is in some
cold case that just doesn't matter. But this isn't even an argument for using
a linked list. This is an argument for using *whatever at all*. Why settle for
a linked list? Use a linked hash map! -->

そうですね！
もしかしたら，あなたのアプリは I/O バウンドになっているのかもしれませんし，
問題のコードはめったに呼び出されないどうでもいい部分かもしれません．
しかし，それは連結リストを使う理由にはなりません．
これは，どんなものでも使うべきだという主張なのです．
連結リストにこだわる理由は何でしょう？
linked hash map (順序つきの辞書) を使えばいいじゃないですか！ 

<!-- If performance doesn't matter, then it's *surely* fine to apply the natural
default of an array. -->

パフォーマンスが問題でないのであれば，配列をそのまま使っても **全然** 問題ないでしょう．

<!-- ## They have O(1) split-append-insert-remove if you have a pointer there -->

## ポインタがそこにあれば，O(1) 時間で分割・追加・挿入・削除が可能

<!-- Yep! Although as [Bjarne Stroustrup notes][bjarne] *this doesn't actually
matter* if the time it takes to get that pointer completely dwarfs the
time it would take to just copy over all the elements in an array (which is
really quite fast). -->

その通りですね！しかし [Bjarne Stroustrup][bjarne] で指摘されているように，この利点は実際には重要ではありません．
配列のすべての要素をコピーするより，連結リストのポインタを取得する方がずっと多くの時間がかかるものだからです．
配列のコピーは実際には非常に高速です．

<!-- Unless you have a workload that is heavily dominated by splitting and merging
costs, the penalty *every other* operation takes due to caching effects and code
complexity will eliminate any theoretical gains. -->

分割や結合のコストが作業負荷の大半を占めるような場合でない限り，
キャッシュ効果やコードの複雑さによって **他のすべて** の操作が受けるペナルティが，あらゆる理論的なメリットを上回ってしまいます．

<!-- *But yes, if you're profiling your application to spend a lot of time in
splitting and merging, you may have gains in a linked list*. -->

しかしまぁ，分割や結合に多くの時間を費やすようなアプリを開発しているのであれば，連結リストにも利点があるかもしれませんね．

<!-- ## I can't afford amortization -->

## 償却の余裕がない

<!-- You've already entered a pretty niche space -- most can afford amortization.
Still, arrays are amortized *in the worst case*. Just because you're using an
array, doesn't mean you have amortized costs. If you can predict how many
elements you're going to store (or even have an upper-bound), you can
pre-reserve all the space you need. In my experience it's *very* common to be
able to predict how many elements you'll need. In Rust in particular, all
iterators provide a `size_hint` for exactly this case. -->

かなりニッチな領域に足を踏み入れているようですね．ほとんどの場合は償却の余裕があります．
確かに，配列は **最悪の場合**, 償却されます．
配列を使用しているからといって，コストも償却されているとは限りません．
もし格納する要素の数が予測できるなら（あるいは上限を決めておけば），必要なスペースをあらかじめ確保することができます．
私の経験では，必要な要素の数を予測できるのは **ごく普通** のことです．
特に Rust では，すべてのイテレータがまさにこのような場合のために `size_hint` メソッドを持っています．

<!-- Then `push` and `pop` will be truly O(1) operations. And they're going to be
*considerably* faster than `push` and `pop` on linked list. You do a pointer
offset, write the bytes, and increment an integer. No need to go to any kind of
allocator. -->

そうすると, `push` と `pop` はまさに O(1) 演算になります．
そして，連結リストの `push` と `pop` よりもかなり高速になります．
ポインタのオフセットを行い，バイトを書き込み，整数をインクリメントします．
アロケータの類を使う必要はありません．

How's that for low latency?

*But yes, if you can't predict your load, there are worst-case
latency savings to be had!*





## Linked lists waste less space

Well, this is complicated. A "standard" array resizing strategy is to grow
or shrink so that at most half the array is empty. This is indeed a lot of
wasted space. Especially in Rust, we don't automatically shrink collections
(it's a waste if you're just going to fill it back up again), so the wastage
can approach infinity!

But this is a worst-case scenario. In the best-case, an array stack only has
three pointers of overhead for the entire array. Basically no overhead.

Linked lists on the other hand unconditionally waste space per element.
A singly-linked list wastes one pointer while a doubly-linked list wastes
two. Unlike an array, the relative wasteage is proportional to the size of
the element. If you have *huge* elements this approaches 0 waste. If you have
tiny elements (say, bytes), then this can be as much as 16x memory overhead
(8x on 32-bit)!

Actually, it's more like 23x (11x on 32-bit) because padding will be added
to the byte to align the whole node's size to a pointer.

This is also assuming the best-case for your allocator: that allocating and
deallocating nodes is being done densely and you're not losing memory to
fragmentation.

*But yes, if you have huge elements, can't predict your load, and have a
decent allocator, there are memory savings to be had!*





## I use linked lists all the time in &lt;functional language&gt;

Great! Linked lists are super elegant to use in functional languages
because you can manipulate them without any mutation, can describe them
recursively, and also work with infinite lists due to the magic of laziness.

Specifically, linked lists are nice because they represent an iteration without
the need for any mutable state. The next step is just visiting the next sublist.

Rust mostly does this kind of thing with [iterators][]. They can be infinite 
and you can map, filter, reverse, and concatenate them just like a functional list,
and it will all be done just as lazily!

Rust also lets you easily talk about sub-arrays with *[slices][]*. Your usual
head/tail split in a functional language is [just `slice.split_at_mut(1)`][split].
For a long time, Rust had an experimental system for pattern matching on
slices which was super cool, but the feature was simplified when it was
stabilized. Still, [basic slice patterns][slice-pats] are neat! And of course,
slices can be turned into iterators!

*But yes, if you're limited to immutable semantics, linked lists can be very
nice*.

Note that I'm not saying that functional programming is necessarily weak or
bad. However it *is* fundamentally semantically limited: you're largely only
allowed to talk about how things *are*, and not how they should be *done*. This
is actually a *feature*, because it enables the compiler to do tons of [exotic
transformations][ghc] and potentially figure out the *best* way to do things
without you having to worry about it. However this comes at the cost of being
*able* to worry about it. There are usually escape hatches, but at some limit
you're just writing procedural code again.

Even in functional languages, you should endeavour to use the appropriate data
structure for the job when you actually need a data structure. Yes,
singly-linked lists are your primary tool for control flow, but they're a
really poor way to actually store a bunch of data and query it.


## Linked lists are great for building concurrent data structures!

Yes! Although writing a concurrent data structure is really a whole different
beast, and isn't something that should be taken lightly. Certainly not something
many people will even *consider* doing. Once one's been written, you're also not
really choosing to use a linked list. You're choosing to use an MPSC queue or
whatever. The implementation strategy is pretty far removed in this case!

*But yes, linked lists are the defacto heroes of the dark world of lock-free
concurrency.*




## Mumble mumble kernel embedded something something intrusive.

It's niche. You're talking about a situation where you're not even using
your language's *runtime*. Is that not a red flag that you're doing something
strange?

It's also wildly unsafe.

*But sure. Build your awesome zero-allocation lists on the stack.*





## Iterators don't get invalidated by unrelated insertions/removals

That's a delicate dance you're playing. Especially if you don't have
a garbage collector. I might argue that your control flow and ownership
patterns are probably a bit too tangled, depending on the details.

*But yes, you can do some really cool crazy stuff with cursors.*





## They're simple and great for teaching!

Well, yeah. You're reading a book dedicated to that premise.
Well, singly-linked lists are pretty simple. Doubly-linked lists
can get kinda gnarly, as we'll see.




# Take a Breath

Ok. That's out of the way. Let's write a bajillion linked lists.

[On to the first chapter!](first.md)


[rust-std-list]: https://doc.rust-lang.org/std/collections/struct.LinkedList.html
[cpp-std-list]: http://en.cppreference.com/w/cpp/container/list
[github]: https://github.com/rust-unofficial/too-many-lists
[bjarne]: https://www.youtube.com/watch?v=YQs6IC-vgmo
[slices]: https://doc.rust-lang.org/std/primitive.slice.html
[split]: https://doc.rust-lang.org/std/primitive.slice.html#method.split_at_mut
[slice-pats]: https://doc.rust-lang.org/edition-guide/rust-2018/slice-patterns.html
[iterators]: https://doc.rust-lang.org/std/iter/trait.Iterator.html
[ghc]: https://wiki.haskell.org/GHC_optimisations#Fusion
[play]: https://play.rust-lang.org/
