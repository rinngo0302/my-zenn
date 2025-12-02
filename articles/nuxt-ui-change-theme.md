---
title: "[Nuxt UI]Nuxt UIのコンポーネントのテーマを設定するには？"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Nuxt, NuxtUI]
published: true
publication_name: "comm_vue_nuxt"
---
# はじめに
最近、あるウェブアプリの開発をするのにNuxt UIを使うことになりました。そこで、めっちゃ便利やなーおもろいなーと思いながら使っていたのですが、少し詰まってしまいました。
それは、「Nuxt UIのコンポーネントの色ってどうやって変更するの？」ということです。
デザインはすでに決まっているのでそれ通りにUIを実装したいのですが、肝心の色を変更する方法がよくわかりませんでした。

そして、いろいろ調べていたのですが、あまり情報もなくあっても情報が古かったり、最近話題のGeminiとかに聞いても情報が少ないのか全然使い物にならなかったりとさんざんな目にあってしまいました。

そこで、私なりにNuxt UIの公式ドキュメントや少しソースコードを見て調べたことを書いておこうと思います。

あと、書いている内容が間違えている可能性は全然ありますのでご了承ください。

# やりたいこと
![](/images/nuxt-ui-change-theme/changedButton.png)

こんな感じで、黄色の背景に黒色のテキストのボタンを作成していきます。

# 環境
- WSL2 Ubuntu 24.04LTS

このテンプレートを使用しています。
https://github.com/shinGangan/nuxt-nuxtui-templates

# コンポーネントのスタイルの設定
公式のドキュメントを読んでいきましょう。
https://ui.nuxt.com/getting-started/theme

Deeplで翻訳したものも置いておきますね。

>(Learn how to customize Nuxt UI components using Tailwind CSS v4, CSS variables and the Tailwind Variants API for powerful and flexible theming.)
> Nuxt UI コンポーネントを Tailwind CSS v4、CSS 変数、および Tailwind Variants API を使用してカスタマイズし、強力で柔軟なテーマ設定を実現する方法を学びます。

Nuxt UIでは基本Taliwind CSSを使っているみたいです。ですので、基本的にはこれをいじってやるとNuxt UIのコンポーネントのスタイルをいじることができそうですね。

> (Nuxt UI uses Tailwind CSS v4, you can read the official upgrade guide to learn about all the breaking changes.
@theme
Tailwind CSS v4 takes a CSS-first configuration approach, you now customize your theme with CSS variables inside a @theme directive to define your project's custom design tokens, like fonts, colors, and breakpoints:
> The @theme directive tells Tailwind to make new utilities and variants available based on these variables. It's the equivalent of the theme.extend key in Tailwind CSS v3 tailwind.config.ts file.)
> 
> Nuxt UI コンポーネントを Tailwind CSS v4、CSS 変数、および Tailwind Variants API を使用してカスタマイズし、強力で柔軟なテーマ設定を実現する方法について学びます。
Tailwind CSS v4 は CSS 優先の構成アプローチを採用しています。@theme ディレクティブ内で CSS 変数を使用してテーマをカスタマイズし、プロジェクトの独自デザイン トークン（フォント、色、ブレイクポイントなど）を定義できます：  
@theme ディレクティブは、これらの変数に基づいて新しいユーティリティとバリアントを Tailwind に提供するように指示します。これは、Tailwind CSS v3 の tailwind.config.ts ファイル内の theme.extend キーに相当します。  

Tailwind CSS **v3**までは、`tailwind.config.ts`という設定ファイルをいじる必要があったのですが、Nuxt UIが採用しているTailwind CSS **v4**ではその設定ファイルをいじらなくても、**CSSベース**でいじることができるみたいです。

そして、それをするために`@theme`を使うといいらしいです。

> (Design system
Nuxt UI extends Tailwind CSS's theming capabilities, providing a flexible design system with pre-configured color aliases based on Tailwind CSS colors. This allows for easy customization and quick adaptation of the UI to your brand's aesthetic.)
> デザインシステム
Nuxt UIはTailwind CSSのテーマ設定機能を拡張し、Tailwind CSSのカラーパレットを基にした事前設定済みのカラーエイリアスを備えた柔軟なデザインシステムを提供します。これにより、ブランドのビジュアルアイデンティティに迅速かつ容易にUIを適応させることが可能です。

Nuxt UIでは、Tailwind CSSのカラーパレットから定義済みのカラーエイリアスを設定することができます。例えば、Tailwind CSSで`indigo`という色を、Nuxt UIの`primary`というカラーエイリアスに設定することができるそうです。

下にデフォルトの設定値を書いておきますね。
|Nuxt UIのカラーエイリアス|Tailwind CSSのカラーパレット|説明|
|-----------------------------|-------------------------------|------------------------------------------------------------------|
|primary                      |green                          |メインブランドカラー。コンポーネントのデフォルトカラーとして使用されます。|
|secondary                    |blue                           |プライマリーカラーを補完するセカンダリーカラー。|
|success                      |green                          |成功状態に使用されます。|
|info                         |blue                           |情報提供状態に使用されます。|
|warning                      |yellow                         |警告状態に使用されます。|
|error                        |red                            |フォームエラーの検証状態に使用されます。|
|neutral                      |slate                          |背景、テキストなどのニュートラルカラー。|

> You can configure these color aliases at runtime in your app.config.ts file under the ui.colors key, allowing for dynamic theme customization without requiring an application rebuild:
> これらのカラーエイリアスは、アプリの設定ファイル（app.config.ts）の ui.colors キーの下で実行時に設定可能です。これにより、アプリケーションの再ビルドを必要とせずに動的なテーマのカスタマイズを実現できます:

`primary`は`green`を設定するという操作は、`app.config.ts`で行うことができます。
書き方については後述します。

あと、各コンポーネントのテーマを設定しているソースコードは以下です。
https://github.com/nuxt/ui/tree/v3/src/theme

例えば、ボタンのテーマを設定している[`button.ts`](https://github.com/nuxt/ui/blob/v3/src/theme/button.ts)を見てみましょう。

Nuxt UIで定義されているカラーエイリアスがありますね。
```ts
const color = [
  "primary",
  "secondary",
  "success",
  "info",
  "warning",
  "error",
  "neutral"
] as const
```

その下あたりに、ボタンの種類を表す定数も定義されていますね。
```ts
const variant = [
  "solid",
  "outline",
  "soft",
  "subtle",
  "ghost",
  "link"
] as const
```

そして、他に定義が終わると次はButtonの設定をしているDictionary型の変数があります。
その中でも１つ抜粋して、`defaultVariants`を見てみましょう。おそらく、名前から考えるにこれがButtonのデフォルトの設定なのでしょう。
```ts
  "defaultVariants": {
    "color": "primary" as typeof color[number],
    "variant": "solid" as typeof variant[number],
    "size": "md" as typeof size[number]
  }
```
このコードを見る限り、`color`に`primary`が設定されていますよね。このようにNuxt UIではコンポーネントの色を設定しています。

# コンポーネントの色を変更するには？
先ほど調査した内容から、以下の手順で変更できると考えられます。
1. `@theme`を使って、Tailwind CSSのカラーパレットを定義する。
2. `app.config.ts`でNuxt UIのカラーエイリアスを設定する。

これで大丈夫だと思います。

# 実際にやってみる
では、UButtonの色を`#ffc000`に設定してみましょう。

Nuxt UIのプロジェクトの`./src/app/assets/css/main.css`を開いてください。

そこで、
```css
@import 'tailwindcss';
@import '@nuxt/ui';

@theme static {
  /* surface色を作成 */
  --color-surface-50: #ffffea;
  --color-surface-100: #fffbc5;
  --color-surface-200: #fff885;
  --color-surface-300: #ffee46;
  --color-surface-400: #ffdf1b;
  --color-surface-500: #ffc000;
  --color-surface-600: #e29400;
  --color-surface-700: #bb6902;
  --color-surface-800: #985108;
  --color-surface-900: #7c420b;
  --color-surface-950: #482200;
}
```
このように書いてください。

この`--color-surface-〇〇`は、CSS変数で、この場合だと`surface`というTailwind CSSのカラーパレットを作成することができます。
そして、この数字は濃度のことを示しています。例えば、ボタンを上でカーソルをホバーさせたとき、少しボタンが薄くなったりしますよね。そういうのを実装するために設定しておきます。
この色は、[Tailwind CSS Color Generator](https://uicolors.app/generate/)などを使うといいと思います。

これで、Tailwind CSSのカラーパレットとして、`surface`を作成することができました。

次に、それをNuxt UIに設定していきましょう。
先ほど紹介したように、`app.config.ts`に設定します。
```ts
export default defineAppConfig({
  ui: {
    colors: {
      primary: 'surface'
    },
    icons: {},
    button: {
      defaultVariants: {
        color: 'primary'
      }
    }
  }
});
```

これで設定できました。
```
primary: 'surface'
```
この部分で設定をしています。

これで実行してみれば、ボタンが黄色になると思います。
また、ほかのコンポーネントも黄色になっていると思うので確認してみてください。

この手順を踏むことで他にもいろいろ設定をすることができます。

# お手軽に一気に変更する
最後に、簡単にぱぱっと変更するやり方を紹介します。
今、UButtonは背景が黄色でテキストは白色なので少し文字が見づらいですよね。
ですので、これを黒色に変更してみましょう。

また、`primary`自体を黄色にしているので、ボタンだけ黒色に設定してもまた他のコンポーネントで設定することになるかもしれないですね。
ですので、全てのテキストを黒色に設定してみましょう。

先ほどの`main.css`を開いてください。
そして、`@theme`の最後に
```css
--ui-text-inverted: var(--color-black);
```
を追加してください。

つまり、
```css
@import 'tailwindcss';
@import '@nuxt/ui';

@theme static {
  /* surface色を作成 */
  --color-surface-50: #ffffea;
  --color-surface-100: #fffbc5;
  --color-surface-200: #fff885;
  --color-surface-300: #ffee46;
  --color-surface-400: #ffdf1b;
  --color-surface-500: #ffc000;
  --color-surface-600: #e29400;
  --color-surface-700: #bb6902;
  --color-surface-800: #985108;
  --color-surface-900: #7c420b;
  --color-surface-950: #482200;

  /* テキストを黒色に設定 */
  --ui-text-inverted: var(--color-black);
}
```
こうなれば大丈夫です。

`--color-black`はすでにTailwind CSSが定義しているカラーパレットで、`--ui-text-unverted`はコンポーネントなどのテキストの色です。
ですので、このように設定すればテキスト全般が一気に黒色になると思います。

# さいごに
私が個人的な開発をするなかで調べた情報を書いてみました。
できるだけ頑張って調べたつもりなのですが、やっぱり基本的に情報源が公式ドキュメントしかなく、少し大変でした...
そのため、間違っていることを書いている可能性がとても高いので、もしなにかあれば指摘していただけるとありがたいです！！

それではーここまで読んでくださりありがとうございましたー