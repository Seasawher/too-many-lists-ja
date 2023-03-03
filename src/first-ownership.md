<!-- # Ownership 101 -->
# 所有権のいろは

<!-- Now that we can construct a list, it'd be nice to be able to *do* something
with it. We do that with "normal" (non-static) methods. Methods are a special
case of function in Rust because of  the `self` argument, which doesn't have
a declared type: -->

さて，リストを作ることができたので，それを使って何かできたらいいですね．
そこで「通常の」(静的ではない) メソッドを使用します．
メソッドは Rust の関数の中でも特殊で，`self` という型を持たない引数を取ります:


<!-- ```rust ,ignore
fn foo(self, arg2: Type2) -> ReturnType {
    // body
}
``` -->

```rust ,ignore
fn foo(self, arg2: Type2) -> ReturnType {
    // 関数の中で行う処理
}
```

<!-- There are 3 primary forms that self can take: `self`, `&mut self`, and `&self`.
These 3 forms represent the three primary forms of ownership in Rust: -->

`self` には，`self`, `&mut self`, `&self` という3つの主要な形があります．
この3つの形態は，それぞれ Rust における所有権の3つの主要な形態に対応しています:

<!-- * `self` - Value
* `&mut self` - mutable reference
* `&self` - shared reference -->

* `self` - 値
* `&mut self` - 可変参照
* `&self` - 共有参照

<!-- A value represents *true* ownership. You can do whatever you want with a value:
move it, destroy it, mutate it, or loan it out via a reference. When you pass
something by value, it's *moved* to the new location. The new location now
owns the value, and the old location can no longer access it. For this reason
most methods don't want `self` -- it would be pretty lame if trying to work with
a list made it go away! -->

値は，**真の** 所有権を表します．移動，破壊，変更，参照による貸し出しなど，値に対してやりたいことは何でもできます．値として何かを渡すと，その値は新しい場所に **ムーブ** されます．
新しい場所がその値を所有することになるので，古い場所はもうその値にアクセスできなくなります．
このため，ほとんどのメソッドは `self` を必要としません．
リストを一度使ったら，もう使えないというのではかなりイライラしますね．

<!-- A mutable reference represents temporary *exclusive access* to a value that you
don't own. You're allowed to do absolutely anything you want to a value you
have a mutable reference to as long you leave it in a valid state when you're
done (it would be rude to the owner otherwise!). This means you can actually completely
overwrite the value. A really useful special case of this is *swapping* a value
out for another, which we'll be using a lot. The only thing you can't do with an
`&mut` is move the value out with no replacement. `&mut self` is great for
methods that want to mutate `self`. -->

可変参照は，自分が所有していない値への一時的で排他的なアクセスを表します．
可変参照している値に対しては，終了時に有効な状態で残しておけば，何をやっても許されます．
(無効にしてしまったら，所有者に対して失礼です！) 
つまり，完全に値を上書きすることもできます．
ある値を別の値に入れ替えることができるので，これは本当に便利です．私たちも後でたくさん使うことになるでしょう．
`&mut` でできないことは，置換せずに値をムーブすることだけです．
`&mut self` は，`self` を変更したいメソッドにぴったりです．

<!-- A shared reference represents temporary *shared access* to a value that you
don't own. Because you have shared access, you're generally not allowed to
mutate anything. Think of `&` as putting the value out on display in a museum.
`&` is great for methods that only want to observe `self`. -->

共有参照は，自分が所有していない値への一時的な共有アクセスを表します．
共有アクセスなので，一般的には何も変更することができません．
これは，値を博物館に展示するようなものだと考えてください．
`&self` は，`self` を観察するだけのメソッドに最適です．

<!-- Later we'll see that the rule about mutation can be bypassed in certain cases.
This is why shared references aren't called *immutable* references. Really,
mutable references could be called *unique* references, but we've found that
relating ownership to mutability gives the right intuition 99% of the time. -->

後に，値の変更に関するルールを迂回できるケースがあることを紹介します．
共有参照が **不変参照** と呼ばれないのはこのためです．
本来なら可変参照も **一意参照** (`訳注`: 原文は unique references) と呼んだ方がいいかもしれませんが，所有権と変性 (`訳注`: 原文は mutability で，値が変更できるかどうかという性質) を結びつけることで 99% のケースで直感的にわかりやすくなります．