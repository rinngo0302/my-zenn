---
title: "[Nuxt UI]UHeaderから学ぶNuxt UIのコンポーネントの仕組み"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Nuxt", "NuxtUI"]
published: false
publication_name: "comm_vue_nuxt"
---
# はじめに
Nuxt UI v4がリリースされて、Nuxt UI Proでしか使えなかったコンポーネントが使えるようになりました。
その中に **UHeader** というとても便利なコンポーネントがあるのでそれを使ってみました。
まずはその紹介と、そこからNuxt UIのコンポーネントがどのように実装されているのかまで紹介しようと思います。

# 環境
**Nuxt UI**: **v4.1.0**
**Nuxt**: v4.2.1
**OS**: macOS 26.1

# UHeaderとは？
まずは基本的なUHeaderの使い方を見ていきましょう。
公式ドキュメントはこちらになります。

https://ui.nuxt.com/docs/components/header

まずはコードを書いてみて、どのようなコンポーネントなのかを見ていきましょう。
```vue
<template>
  <UHeader title="UHeader" />
</template>
```
これで見てみましょう。

PC用では
![](https://storage.googleapis.com/zenn-user-upload/16b69e70f1ac-20251202.png)

スマホ用では
![](https://storage.googleapis.com/zenn-user-upload/d467cde7782e-20251202.png)

こんな感じですね。
それではここから設定できる値を見てみましょう。

## 属性だけで設定する
UHeaderはとても便利なコンポーネントで、属性を設定するだけでUIを調整することができます。

### `title`属性
さっきのコードにも含まれていたと思いますが、名前の通り`title`という属性を使うことでヘッダーのタイトルを設定することができます。

例えば、`title="Hello"`にしてみましょう。
```vue
<template>
  <UHeader title="Hello" />
</template>
```
そうすると、先ほど`UHeader`だったのが`Hello`に変わりましたよね。

![](https://storage.googleapis.com/zenn-user-upload/21b27d064107-20251202.png)

このように簡単にタイトルを設定することができます。

### `to`属性
次に、タイトルをクリックしたときの遷移先を設定することができます。
Webページでタイトルをクリックするとホームに飛ぶというような実装をよく見かけますよね。
それです。

では、
```vue
<template>
  <UHeader title="UHeader" to="https://ui.nuxt.com/docs/components/header" />
</template>
```
このようなコードに変更してみましょう。

そしてタイトルを押すと、Nuxt UIのUHeaderのドキュメントに飛びましたよね。
他にも好きなURLを設定してみてください。そうするとそのWebサイトに飛ぶようになると思います。

### `mode`属性
次に、ヘッダーの展開方法を設定してみます。
まずは現在の設定を確認してみましょう。

コードは先ほどのままで大丈夫です。
```vue
<script setup lang="ts">
import type { NavigationMenuItem } from '@nuxt/ui';

const items = ref<NavigationMenuItem[]>([
  {
    label: 'ページ1',
    to: 'https://ui.nuxt.com/docs/components/header'
  },
  {
    label: 'ページ2',
    to: 'https://ui.nuxt.com/docs/components/header'
  },
  {
    label: 'ページ3',
    to: 'https://ui.nuxt.com/docs/components/header'
  }
]);
</script>

<template>
  <UHeader title="UHeader">
    <template #body>
      <UNavigationMenu :items="items" orientation="vertical" />
    </template>
  </UHeader>
</template>
```

では、ブラウザの横幅を小さくするか開発者モードでスマホ用にしてみましょう。
すると下のスクショのように **ハンバーガーメニュー** が出てきましたよね。

![](https://storage.googleapis.com/zenn-user-upload/166023138071-20251202.png)

これをクリックしてみてください。
すると、ページ一覧のようなものが出てきました。
![](https://storage.googleapis.com/zenn-user-upload/a8f9c87e8080-20251202.png)

`NavigationMenu`というコンポーネントが置いてあります。
https://ui.nuxt.com/docs/components/navigation-menu

このハンバーガーメニューを押すとこれが表示されるようになっているんですね。

注目してほしいのは、これが切り替わる瞬間です。
パッとアニメーションもなにもせずに切り替わってますよね。

今から使う`mode`という属性を使用することで、この出てき方を変えることができます。

では、ソースコードを以下のように変更してみましょう。といっても、`mode`の値を書き換えるだけですね。
```vue
<script setup lang="ts">
import type { NavigationMenuItem } from '@nuxt/ui';

const items = ref<NavigationMenuItem[]>([
  {
    label: 'ページ1',
    to: 'https://ui.nuxt.com/docs/components/header'
  },
  {
    label: 'ページ2',
    to: 'https://ui.nuxt.com/docs/components/header'
  },
  {
    label: 'ページ3',
    to: 'https://ui.nuxt.com/docs/components/header'
  }
]);
</script>

<template>
  <UHeader title="UHeader" mode="slideover">
    <template #body>
      <UNavigationMenu :items="items" orientation="vertical" />
    </template>
  </UHeader>
</template>
```
すると、背景が薄くなり左から出てくるようなアニメーションになりました。
![](https://storage.googleapis.com/zenn-user-upload/9de0e0fbae7f-20251202.png)

この設定値は3つあり、
|設定値|効果|
|------|-----|
|`modal`|パッと切り替わる|
|`slideover`|背景が薄くなり、左から出てくる|
|`drawer`|背景が薄くなり、下から出てくる|

とりあえず試してみるといいと思います。

## `<template>`を使ってみる
次に`template`というものを使ってみましょう。
これを使うことで、要素をより柔軟に設置することができます。

それでは先ほどのコードを見てみましょうか。
```vue
<script setup lang="ts">
import type { NavigationMenuItem } from '@nuxt/ui';

const items = ref<NavigationMenuItem[]>([
  {
    label: 'ページ1',
    to: 'https://ui.nuxt.com/docs/components/header'
  },
  {
    label: 'ページ2',
    to: 'https://ui.nuxt.com/docs/components/header'
  },
  {
    label: 'ページ3',
    to: 'https://ui.nuxt.com/docs/components/header'
  }
]);
</script>

<template>
  <UHeader title="UHeader" mode="slideover">
    <template #body>
      <UNavigationMenu :items="items" orientation="vertical" />
    </template>
  </UHeader>
</template>
```
これを見る限り、`<template>`を使用している箇所は2つありますよね。

- 全体の要素
- `UHeader`の中の`<template #body>`

今回使うのは後者のUHeaderの中に入っている方です。

では、試しに`<template #body>`の中身を書き換えてみましょう。
```vue
<script setup lang="ts">
import type { NavigationMenuItem } from '@nuxt/ui';

const items = ref<NavigationMenuItem[]>([
  {
    label: 'ページ1',
    to: 'https://ui.nuxt.com/docs/components/header'
  },
  {
    label: 'ページ2',
    to: 'https://ui.nuxt.com/docs/components/header'
  },
  {
    label: 'ページ3',
    to: 'https://ui.nuxt.com/docs/components/header'
  }
]);
</script>

<template>
  <UHeader title="UHeader" mode="slideover">
    <template #body>
      <UButton label="ボタン" />    <!-- ボタンを追加 -->
      <UNavigationMenu :items="items" orientation="vertical" />
    </template>
  </UHeader>
</template>
```
とりあえず`UButton`を追加してみました。

では、ハンバーガーメニューを見てみましょう。
ボタンが追加されましたよね。
![](https://storage.googleapis.com/zenn-user-upload/a2d26cfa3ba4-20251202.png)

こんな感じで、`<template #body>`の中に要素を追加することでハンバーガーメニューの中身を実装することができます。

ちなみにこれ`body`ですが、他にも
- top
- bottom
- left
- right

が用意されています。
これらはハンバーガーメニューではなくヘッダー自身に設定するものです。

# 詰まった点
さて、ここまで基本的なUHeaderの使い方を見てきましたが、結構便利だなーと思ったことでしょう。
はい、すごい便利なんですが...1つだけ問題があります...

それは、
**「ハンバーガーメニューがスマホでしか見えない」**
というものです。

おそらく、PC版では普通に要素を出しておいて、スマホ版では画面が小さいからハンバーガーメニューを用意してそこで表示してねっていう意味だと思います。
ですが、PC版でも使いたいときがありますよね？

私は、これはちょっと困るのでは？というかこれバグなのでは？と思い、いろいろ調査することにしました。
そして、これを調査するうえでNuxt UIのコンポーネントの仕組みについても知ることができたのでそれも紹介しようと思います。

# UHeaderの仕組みを調査する
それでは調査していきましょう。
目標は、 

**「ハンバーガーメニューの表示非表示の仕組みを見る」**

です。


## UHeaderの定義ファイルを探す
Nuxt UIのGitHubから探していきましょう！

https://github.com/nuxt/ui/tree/v4

コンポーネントなので、おそらく`*.vue`ですよね。そして、UHeaderなので`header.vue`とかなんじゃないでしょうか？
とりあえずGitHub上で検索してみましょう。

![](https://storage.googleapis.com/zenn-user-upload/f0dd199fd155-20251125.png)

おそらく一番上の `src/runtime/components/Header.vue` が怪しそうですよね。
中を見てみて、とりあえず`<template></template>`で囲っている中を見てみると、なーんとなくこれはヘッダーっぽいですよね。
```vue:src/runtime/components/Header.vue
<template>
  <DefineToggleTemplate>
    <slot name="toggle" :open="open" :toggle="toggleOpen" :ui="ui">
      <UButton
        v-if="toggle"
        color="neutral"
        variant="ghost"
        :aria-label="open ? t('header.close') : t('header.open')"
        :icon="open ? appConfig.ui.icons.close : appConfig.ui.icons.menu"
        v-bind="(typeof toggle === 'object' ? toggle as Partial<ButtonProps> : {})"
        data-slot="toggle"
        :class="ui.toggle({ class: props.ui?.toggle, toggleSide })"
        @click="toggleOpen"
      />
    </slot>
  </DefineToggleTemplate>

  <DefineLeftTemplate>
    <div data-slot="left" :class="ui.left({ class: props.ui?.left })">
      <ReuseToggleTemplate v-if="toggleSide === 'left'" />

      <slot name="left">
        <ULink :to="to" :aria-label="ariaLabel" data-slot="title" :class="ui.title({ class: props.ui?.title })">
          <slot name="title">
            {{ title }}
          </slot>
        </ULink>
      </slot>
    </div>
  </DefineLeftTemplate>

  <DefineRightTemplate>
    <div data-slot="right" :class="ui.right({ class: props.ui?.right })">
      <slot name="right" />

      <ReuseToggleTemplate v-if="toggleSide === 'right'" />
    </div>
  </DefineRightTemplate>

  <Primitive :as="as" v-bind="$attrs" data-slot="root" :class="ui.root({ class: [props.ui?.root, props.class] })">
    <slot name="top" />

    <UContainer data-slot="container" :class="ui.container({ class: props.ui?.container })">
      <ReuseLeftTemplate />

      <div data-slot="center" :class="ui.center({ class: props.ui?.center })">
        <slot />
      </div>

      <ReuseRightTemplate />
    </UContainer>

    <slot name="bottom" />
  </Primitive>

  <Menu
    v-model:open="open"
    :title="t('header.title')"
    :description="t('header.description')"
    v-bind="menuProps"
    :ui="{
      overlay: ui.overlay({ class: props.ui?.overlay }),
      content: ui.content({ class: props.ui?.content })
    }"
  >
    <template #content="contentData">
      <slot name="content" v-bind="contentData">
        <div v-if="mode !== 'drawer'" data-slot="header" :class="ui.header({ class: props.ui?.header })">
          <ReuseLeftTemplate />

          <ReuseRightTemplate />
        </div>

        <div data-slot="body" :class="ui.body({ class: props.ui?.body })">
          <slot name="body" />
        </div>
      </slot>
    </template>
  </Menu>
</template>
```

ですので、とりあえずこれがUHeaderを定義しているファイルということで間違いないでしょう。
では、これからこのファイルを詳細に見ていくことにしましょう。

:::message
Nuxt UIのコンポーネントは、[`src/runtime/components`](https://github.com/nuxt/ui/tree/v4/src/runtime/components)で定義されている。
:::


## ハンバーガーメニューの挙動を考える
とりあえずハンバーガーメニューの挙動を見るために、ハンバーガーメニューのボタンを探してみましょう。
といっても、このファイルの中に`UButton`は１つしかないですよね。
```vue
  <DefineToggleTemplate>
    <slot name="toggle" :open="open" :toggle="toggleOpen" :ui="ui">
      <UButton
        v-if="toggle"
        color="neutral"
        variant="ghost"
        :aria-label="open ? t('header.close') : t('header.open')"
        :icon="open ? appConfig.ui.icons.close : appConfig.ui.icons.menu"
        v-bind="(typeof toggle === 'object' ? toggle as Partial<ButtonProps> : {})"
        data-slot="toggle"
        :class="ui.toggle({ class: props.ui?.toggle, toggleSide })"
        @click="toggleOpen"
      />
    </slot>
  </DefineToggleTemplate>
```
`UHeader`にはボタンが１つしかないのでおそらくこの部分ですね。
よく見てみると、
```vue
v-if="toggle"
```
と書いてあります。つまり`toggle`がtrueかfalseかでこのボタンを表示非表示を決めているみたいですね。

では`toggle`がどうやって値が設定されているのか見てみましょう。

これはPropsから渡ってきてるので`props`を見てみましょう。
```ts
const props = withDefaults(defineProps<HeaderProps<T>>(), {
  as: 'header',
  mode: 'modal' as never,
  toggle: true,
  toggleSide: 'right',
  to: '/',
  title: 'Nuxt UI'
})
```
これを見てみると`toggle: true`があります。
つまり、**デフォルトでは表示されている** ということがわかりました！

そして、これを表示非表示は自分で決めることができ、
```vue
<UHeader :toggle="false" />
```
にすると非表示にできることがわかりました。

![](https://storage.googleapis.com/zenn-user-upload/b4bd6c84bed7-20251202.png)

ですが、これではスマホとPCで表示がなぜ違うのかわかりませんね。そもそもこのコードには画面の横幅を認識するようなコードが書かれていません。
ですので、次はスタイルが記述されているところを見に行きましょう。

このコードを見てみると、
```vue
import theme from '#build/ui/header'
```
というコードがあります。ですのでこれを見てみましょう。

## デザインファイルを見にいく
まずデザインファイルを探しにいくのですが、その前にこのハンバーガーメニューのクラスを確認しておきましょう。基本スタイルはクラスを使って指定されるので。

`UButton`のコードはこうでしたよね。
```vue
      <UButton
        v-if="toggle"
        color="neutral"
        variant="ghost"
        :aria-label="open ? t('header.close') : t('header.open')"
        :icon="open ? appConfig.ui.icons.close : appConfig.ui.icons.menu"
        v-bind="(typeof toggle === 'object' ? toggle as Partial<ButtonProps> : {})"
        data-slot="toggle"
        :class="ui.toggle({ class: props.ui?.toggle, toggleSide })"
        @click="toggleOpen"
      />
```
すると、
```vue
:class="ui.toggle({ class: props.ui?.toggle, toggleSide })"
```
と書かれているみたいです。これを覚えておいてくださいね。

では、ファイルを探しましょう。
といっても、`#build/ui/header`というヒントがあるのでこれを手掛かりにしましょう。

まずは`src`フォルダですね。
![](https://storage.googleapis.com/zenn-user-upload/b40989ec2dba-20251125.png)

次に、`theme`っていういかにもなフォルダがありますよね。
これをのぞいてみましょう。
![](https://storage.googleapis.com/zenn-user-upload/1bd6946a8d56-20251125.png)

すると、コンポーネントの名前が書かれたファイルがたくさん出てくると思うので、この中からheaderが書かれているものを探しましょう。
![](https://storage.googleapis.com/zenn-user-upload/d1b01fb1287f-20251125.png)

開いてみると、TailwindCSSが記述されていることがわかります。

```ts
export default {
  slots: {
    root: 'bg-default/75 backdrop-blur border-b border-default h-(--ui-header-height) sticky top-0 z-50',
    container: 'flex items-center justify-between gap-3 h-full',
    left: 'lg:flex-1 flex items-center gap-1.5',
    center: 'hidden lg:flex',
    right: 'flex items-center justify-end lg:flex-1 gap-1.5',
    title: 'shrink-0 font-bold text-xl text-highlighted flex items-end gap-1.5',
    toggle: 'lg:hidden',
    content: 'lg:hidden',
    overlay: 'lg:hidden',
    header: 'px-4 sm:px-6 h-(--ui-header-height) shrink-0 flex items-center justify-between gap-3',
    body: 'p-4 sm:p-6 overflow-y-auto'
  },
  variants: {
    toggleSide: {
      left: {
        toggle: '-ms-1.5'
      },
      right: {
        toggle: '-me-1.5'
      }
    }
  }
}
```

そして、よく見てみると
```css
toggle: 'lg:hidden'
```
と記述されています。
先ほど覚えてもらったクラスにも`toggle`が書かれていましたよね。

つまり、ハンバーガーメニューは`lg:hidden`というスタイルが適用されていることになります。
そして`lg:hidden`は画面が1024px以上になると要素を消すというものです。

つまり、これがあることでPC版はハンバーガーメニューが消えていたんですね。

# 解決策
結論として、スタイルシートにPC版だと消えるように設定されていることがわかりました。
これで私の目標にしていた
**「ハンバーガーメニューの表示非表示の仕組みを見る」**
は達成できました。

最後に解決策を紹介しておきましょう。
スタイルシートで完全に設定されてしまっているので、もうこのハンバーガーメニューを消してしまって、自分でボタンとハンバーガーメニューの要素を置くためのコンポーネントを用意するしかないです。

`header.vue`を見てみると、
```ts
const Menu = computed(() => ({
  slideover: USlideover,
  modal: UModal,
  drawer: UDrawer
})[props.mode as HeaderMode])
```
こんな感じで、`USlideover`と`UModal`、`UDrawer`が使われているみたいなのでこれを自分で用意してあげるとできると思います。

# まとめ
それでは最後に、このようにコンポーネントが思った通りにうまくいかなかったときにどうするべきかをおさらいしておきましょう。

私は以下の順番で進めるといいと思います。
1. [Nuxt UIのドキュメント](https://ui.nuxt.com/docs/getting-started)を読む
2. [GitHubのリポジトリ](https://github.com/nuxt/ui)を見にいく
3. [`src/runtime/components`](https://github.com/nuxt/ui/tree/v4/src/runtime/components)の中を見にいく
4. デザイン関連なら[`src/theme`](https://github.com/nuxt/ui/tree/v4/src/theme)の中を見にいく

とりあえずこれを試してみましょう！

# さいごに
この記事では`UHeader`でNuxt UIのコンポーネントの仕組みについてと、合わせてドキュメントを読んだだけではわからないときのソースコードの読み方について解説しました。

Nuxt UIの記事はまだまだ少なく、ドキュメントかソースコードを読みに行くしか手段がないため少し勉強しづらいところはあります。
ですので、そのような状況になったときにどのように解決すべきかを考える手助けができればいいと思います。