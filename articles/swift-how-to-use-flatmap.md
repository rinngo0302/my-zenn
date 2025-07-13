---
title: "[Swift] flatMapのメモ"
emoji: "🍎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Swift"]
published: true
---

# はじめに
バイトでSwiftを使っているんですけど、そこで`flatMap`というものを初めて知りました。
それで少し勉強したんですけど、きちんと理解していないところもあるので、もう少し勉強してみてそれをメモおこうと思います！

# Optional.flatMapとは？
`Optional.flatMap`とは、変数が`nil`ならそのまま`nil`を返し、そうでなければそのクロージャを実行して値を返すというものです。

よく、クロージャの中では変数を**非オプショナル型として扱う必要があるとき**に使われることが多いです。

https://developer.apple.com/documentation/swift/optional/flatmap(_:)

# Optional.flatMapの使い方
それでは、どのように使うのかを実際に例を出して紹介してみます。

まずは以下のような関数を用意します。

```Swift
func calc(num: Int) -> Int {
    num * 2
}
```

この関数は、非オプショナル型（`Int`型）の引数を受け取り、その値を2倍にして返します。

次に、以下のようなオプショナル型の変数を定義します。

```Swift
let num: Int? = 5
```

この変数をそのまま`calc`関数に渡して

```Swift
let result = calc(num: num)
```

とすると

```
Value of optional type 'Int?' must be unwrapped to a value of type 'Int'
```

とエラーが出ます。
:::details エラーについて
このエラーは`calc`関数が非オプショナル型（`Int`型）を引数に取るのに対し、`num`がオプショナル型（`Int?`型）であるので型が一致しないというものです。
Swiftでは、オプショナル型をそのまま非オプショナル型の引数に渡すことはできません。
:::

この程度なら、例えば
```Swift
let result = calc(num: num ?? 0)
```
にして実行できますが、これでは`result`が`nil`であることを検知することができません。
できれば`num`が`nil`の時は`result`も`nil`にしたいですよね。

その場合に便利なのが`flatMap`です。
```Swift
let result: Int? = num.flatMap{ calc(num: $0) }
```
このコードでは、`flatMap`を使うことで以下の動作を実現しています：
1. `num`が`nil`の場合、クロージャは実行されず、`result`には`nil`が渡されます。
2. `num`が`nil`でない場合、`calc`関数が実行され、その返り値が`result`に渡されます。

このように、`flatMap`を使うことで、オプショナル型を安全にアンラップしつつ、クロージャ内で処理を行うことができます。

:::message
**Optional.flatMapは**
```Swift
変数.flatMap{ 
    クロージャ
    // 変数は $0 として扱える
}
```
で使うことができ、変数が`nil`ではないときにクロージャが実行されます。
:::

# Array.flatMapとは？
`swift flatMap`とかで検索していると、配列バージョンの`Array.flatMap`も出てきます。

このメソッドは、要素を変換しつつ、ネストされた配列を一次元配列に変換することができます。  
例えば、
```Swift
let array: [[Int]] = [
    [1, 2, 3],
    [4, 5],
    [6, 7, 8],
    [],
    [9]
]
```
を
```Swift
let flatArray = array.flatMap { $0 }
```
で、以下のような一次元配列になります。

```Swift
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

https://developer.apple.com/documentation/swift/array/flatmap(_:)-i3mr

このように、今回のテーマである`Optional.flatMap`と`Array.flatMap`は名前こそ似ているものの、**全然違うもの**なので、注意してください！

# さいごに
今回は、`Optional.flatMap`についての記事を書きました。
バイトをしているときに知って試しに調べてみたものですが、思ったよりも便利なメソッドだったので調べて良かったなと思います。

まだまだSwiftは勉強中ですが、これからもいろいろ勉強していきます！