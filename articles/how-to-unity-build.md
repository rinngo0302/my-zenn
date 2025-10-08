---
title: "[Unity]Unityでビルドする方法"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [unity]
published: true
---
# はじめに
Unityでビルドっていろいろあって少しわかりづらいですよね？
この記事では、初心者の方のために**Unityのビルド** の設定などについて解説しようと思います。

# 環境
**OS**: macOS 26.01
**Unity**: Unity 6000.2.6f2

今回はMacを使っていますが、**Windowsでも操作はほとんど変わらない**のでご安心ください。

# ビルドとは？
Unityを使って開発をしてきて、ようやくリリースするタイミングになると、**ビルド**という作業をする必要があります。
この作業をすることで、今ではUnityエディタ上でしか遊べなかったのが、他の人でも遊べるようになったり**UnityRoom**に公開することもできます。

# ビルド方法
ビルドする手順として
1. シーンを設定する
2. ビルドするプラットフォームを設定する
3. ビルドする

## シーンを設定する
まずはゲームをプレイしたときの**一番最初のシーン**を設定しましょう。
例えば、ゲームならタイトルシーンを一番最初にしたいですよね。

といっても、やり方は結構簡単なんです。

まずは画像のように`Build Profiles`をクリックしてください。

![](https://storage.googleapis.com/zenn-user-upload/53b88f1d8713-20251007.png)

その後、出てきたウィンドウの左の`Scene List`をクリックしてください。
そうすると、このような画面が出てくると思います。

![](https://storage.googleapis.com/zenn-user-upload/9a8e7eef8fc4-20251007.png)

そして、**一番最初のシーン**を設定するのは簡単で、**Scene Listで一番最初にしたシーン**が一番最初に表示されます。

今回だと、`TitleScene`を一番最初にしたいので、この画像のように`TitleScene`を一番上にすれば大丈夫です。

![](https://storage.googleapis.com/zenn-user-upload/c18a57f8961a-20251007.png)

## ビルドするプラットフォームを設定する
次にどのプラットフォーム用にビルドをするかを選択します。
プラットフォームというのは、Windows上なのかUnityRoom用なのか...ということですので、もし「Windows用にゲームを作りたい」なら**Windowsをビルドするプラットフォームとして選択**する必要があります。

この記事では、一番使うであろう「Windows用」と「UnityRoom用」の２つのやり方を解説します。

### Windows用にビルドする
先ほどの`Build Profiles`のウィンドウの、左の **「Platforms」**から「Windows」を選択してください。

![](https://storage.googleapis.com/zenn-user-upload/5bd5247d8581-20251008.png)

もしかしたらUnityのバージョンによったら画面構成は変わるかもしれないのですが、ほとんど同じですので気にしなくて大丈夫です。

そして、Windowsでビルドする場合はおそらくデフォルトの状態でビルドできますので、下のスクショのように **「Build」** を押してください。

![](https://storage.googleapis.com/zenn-user-upload/8ba961982a24-20251008.png)

それであとはビルドファイルを置く場所を設定して、あとはビルドを待てば完了します。

### UnityRoom用にビルドする
こちらはWindowsと違って少しプロジェクトの設定をする必要があります、

上のメニューから「Edit」->「Project Settings」をクリックしてください。
(もしかしたら、古いバージョンでは「Project Properties」になっているかもしれないです。)

そして、左のところから **「Player」** を選択してください。

![](https://storage.googleapis.com/zenn-user-upload/0ffe75d2b082-20251008.png)

次に **「Publishing Settings」** を押してください。

![](https://storage.googleapis.com/zenn-user-upload/de7eac04f519-20251008.png)

すると **「Compression Format」** の項目で **「Gzip」** という選択肢があるのでそれを選択してください。

![](https://storage.googleapis.com/zenn-user-upload/e8ad135d60e6-20251008.png)

![](https://storage.googleapis.com/zenn-user-upload/ac5e8f422baa-20251008.png)

先ほどの`Build Profiles`のウィンドウの、左の **「Platforms」** から「Web」を選択してください。

そうするとしたの画像のように、デフォルトのままでは **「No Web module loaded」** と表示されていると思います。

![](https://storage.googleapis.com/zenn-user-upload/bf36c4c96d5b-20251008.png)

これは、UnityRoom用のビルドするためのプログラムが入っていないという意味です。
ですので、それをインストールしてみましょう。

下のスクショのように **「Install with Unity Hub」** というボタンを押してください。

![](https://storage.googleapis.com/zenn-user-upload/ee40e08b503d-20251008.png)

するとこのようなウィンドウが出てきます。
そのまま **「インストール」** を押すとインストールが始まります。

![](https://storage.googleapis.com/zenn-user-upload/a9cd359c2322-20251008.png)

それが終わったら、一度Unityエディタを再起動しましょう。
そして先ほどと同じ「Build Profiles」の「Web」を開いてください。

すると先ほどと違ってこのようにいろいろ書かれていますよね。
そうなるとインストールは成功しています。

ですが、この状態まだビルドをすることはできません。
なぜなら、この状態では選択されているプラットフォームがデフォルトの状態になっている可能性があるからです。

もし、下の画像のように「Build」が灰色になって押せなくなっているときは、右の **「Switch Platform」** というボタンをクリックしてください。

![](https://storage.googleapis.com/zenn-user-upload/a47dde971b0d-20251008.png)

そうすると、画像のように「Web」のところに「Active」と書いてあれば大丈夫です。
この状態で、 **「Build」** を押してください。

![](https://storage.googleapis.com/zenn-user-upload/a5f07f691dea-20251008.png)

これでビルドファイルを置く場所を設定すれば大丈夫です。

# おわりに
この記事では忘れやすいビルド方法についてまとめました。
もしかしたらUnityのバージョンによってボタンの位置とかが変わっているかもしれないのですが、基本的には同じ画面の中にあると思うので根気よく探してください。

それではお疲れ様でしたー