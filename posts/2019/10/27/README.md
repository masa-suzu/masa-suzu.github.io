# プログラミング言語「Koka」で遊ぶ

`Algebraic effects and Handlers`を実装した言語であるKokaで遊んだ。

{% gist 3bef8f1ecfac144a50a21cf5fcd5f68d %}

一般的なプログラミング言語では馴染みがあるイテレータと例外構文を実装した。

## 動作環境

- [ソースコード](https://github.com/koka-lang/koka/commit/ff10f073dec3f92d412996f7bb3d3dc72e9cead8)を[Dockerコンテナ上](https://github.com/masa-suzu/dev-env/blob/master/koka/dockerfile)でビルドした。

## effectとhandler

- effect

- handler

### effect

一般的なインターフェースのような定義のような定義になっている。
```
public effect enumerable<t> {
    fun yield(item:t) : ()
}

public effect panic<t> {
    fun happen(s:string) : ()
}
```

例えば、`yield`関数は


- 任意の型`a`

について、

 - `a`の値`item`

を受け取り

- `()`を発生させる効果`enumerable<a> ()`

を返す関数になっている。

```
> :t yield
forall<a>. (item : a) -> (examples/enumerable<a>) ()
```

### handler

発生した効果をどのように処理するのかを定義する。効果ごとにハンドラを記述する。
```
public fun foreach(action, f) {
    handle(action) {
        yield(x) -> { f(x); resume(()) }
    }
}

public fun try_catch(action, f) {
    handle(action) {
        happen(e) -> f(e)
    }
}

public fun try_continue(action, f) {
    handle(action) {
        happen(e) -> { f(e); resume(()) }
    }
}
```

例えば、`foreach`関数は、

- 任意の型`a,b,c`
- 効果`e`

について、

 - `b`を発生させる効果`examples/enumerable<a>`あるいは効果`e`を返す関数`action`
 - `c`を発生させる効果`e`を返す関数`f`

を受け取り

- `b`を発生させる効果`e`

を返す関数になっている。

```
:t examples/foreach
forall<a,b,c,e>. (action : () -> <examples/enumerable<a>|e> b, f : (a) -> e c) -> e b
```

`action`が`examples/enumerable<a>|e`になっており、`enumerable<a>`でない効果を上位のハンドラの処理を上位のハンドラに移譲できるようになっている。

キーワード`resume`によって、継続を再開できる。`yield`関数の戻り値の型は`()`なので、`resume(())`のように再開する。

`try_catch`ハンドラでは、継続を再開しないので、`resume(())`を使用していない。


## 具体的な計算

- effect `enumerable`,`panic`

- handler `foreach`,`try_catch`,`try_continue`

を使用して、具体的な計算を行う。

```
public fun main() {

    try_catch({
        foreach({do()}, fun(s:string) {s.println})
    }, found_panic)

    breakLines()

    foreach({
        try_continue({do()}, found_panic)
    },  fun(s:string) {s.println})
}

public fun do() {
    iterate(["1", "2", "3"])
    happen("oops")
    iterate(["4", "5", "6"])
}

fun found_panic(s:string) {
    val message = "found a panic: " + s
    message.println
}

fun breakLines(){
    "".println
}
```

ここで、関数`main`と`do`の型はそれぞれ以下のようになっている。
```
> :t do
forall<a>. () -> <examples/enumerable<string>,examples/panic<a>> ()

> :t main
() -> console ()
```

`main`関数を実行すると、iterateにより列挙されたアイテムをハンドルしならが、panicの処理を実施している。

- `try_catch`の場合は、`panic`のハンドル後に処理の再開はしない。

- `try_continue`の場合は、`panic`のハンドル後に処理を再開している。

```
> :l examples.kk
...

> main()
1
2
3
found a panic: oops

1
2
3
found a panic: oops
4
5
6

```

## References

- [Misreading Chat #63: Programming with Algebraic Effects and Handlers](https://misreading.chat/2019/06/23/episode-63-programming-with-algebraic-effects-and-handlers/)

- [Algebraic Effects for Functional Programming
](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/08/algeff-tr-2016-v3.pdf)

- [Algebraic Effectsとは? 出身は? 使い方は? その特徴とは? 調べてみました!](https://qiita.com/Nymphium/items/e6ce580da8b87ded912b)