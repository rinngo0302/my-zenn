---
title: "[Nuxt]Propsのメモ"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Nuxt"]
published: false
---
# はじめに
最近、Nuxtを勉強することになったので、そのついでに気になったことをメモをしていきながら勉強していこうと思います！！

というわけで、NuxtはVueを使うので、まずはVueの`Props`を確認していこーかなと思いまーす！

なお、この記事は[Nuxt 3　フロントエンド開発の教科書](https://amzn.asia/d/fzsTyVH)を参考にしております。

# Propsとは？
Vueには、**Props**という機能があります。
これはデータを**親のコンポーネントから子のコンポーネントへ渡す**ものです。

ちょっと分かりづらいので、図で考えてみます。

![](https://storage.googleapis.com/zenn-user-upload/475b6e85c7c4-20250427.png)

上の図では、親コンポーネントが子コンポーネントにデータを渡している様子を示しています。`Props`を使うことで、親から子にデータを簡単に渡すことができます。

## いつ使うの？
まずは、適当なコンポーネントを用意してみました。
![](https://storage.googleapis.com/zenn-user-upload/d08f9adff42d-20250427.png)
ソースコードはこちらです。

これが子コンポーネントです。
```vue: components/UserCard.vue
<template>
  <div class="userCard">
    <h2> 田中太郎 </h2>
    <p> ID: 1 </p>
    <p> email: taro.tanaka@example.com </p>
  </div>
</template>

<style scoped>
.userCard {
  border: 1px solid green;
}
</style>
```

こっちは親コンポーネントです。
```vue: app.vue
<template>
  <div>
    <UserCard />
    <UserCard />
  </div>
</template>
```

実際に動かしてもらえば分かると思うのですが、本当にただのコンポーネントです。
ですが、このままだと田中太郎さんのデータしか表示されず、他の人のデータを表示することができないですよね？

これを解決するのが**Props**です。

親コンポーネントにあるデータを子コンポーネントに渡すことでいろいろなデータを表示することができるんです。

## Propsを送信する
まずは、データを渡すためにそのデータの構造を定義します。ここでは`interface`を使って型を定義します。

```ts: app.vue
interface Member {
  name: string;
  id: number;
  email: string;
  note?: string;
}
```

こんな感じでいいですかね。

それでは`app.vue`に追加しましょう。
```vue: app.vue
<script setup lang="ts">
interface Member {
  name: string;
  id: number;
  email: string;
  note?: string;
}
</script>

<template>
  <div>
    <UserCard />
    <UserCard />
  </div>
</template>
```

では、この`Member`型のインスタンスをつくっていきましょう。
```ts: app.vue
const memberListInit = new Map<number, Member>();
memberListInit.set(1, {
  id: 1,
  name: '田中太郎',
  email: 'taro.tanaka@example.com',
  note: 'ノートです。'
});
memberListInit.set(2, {
  id: 2,
  name: '鈴木次郎',
  email: 'suzuki.ziro@example.com'
});
```
こんな感じで大丈夫そうですね。

では、値が変わったらUIも自動で変わるようにするのに
```ts: app.vue
const memberList = ref(memberListInit);
```
`ref`を実装しましょう。

それでは、一旦実装したものを出しますね。
```vue: app.vue
<script setup lang="ts">
interface Member {
  name: string;
  id: number;
  email: string;
  note?: string;
}

const memberListInit = new Map<number, Member>();
memberListInit.set(1, {
  id: 1,
  name: '田中太郎',
  email: 'taro.tanaka@example.com',
  note: 'ノートです。'
});
memberListInit.set(2, {
  id: 2,
  name: '鈴木次郎',
  email: 'suzuki.ziro@example.com'
});

const memberList = ref(memberListInit);
</script>

<template>
  <div>
    <UserCard />
    <UserCard />
  </div>
</template>
```

それでは次にPropsを送信していきましょう。
送信するには`v-bind`を使います。そして、これは省略して`:`で書けるので
```vue: app.vue
<UserCard :name="member.name">
```
このように書くとPropsを送信することができます。

それでは実装していきましょう。
なお、app.vueの中でデータは二つ以上あるので`v-for`も使いましょう。
```vue: app.vue
<template>
  <div>
    <UserCard
      v-for="[id, member] in memberList"
      :id="id"
      :key="id"
      :name="member.name"
      :email="member.email"
      :note="member.note"
    />
  </div>
</template>
```
これで子コンポーネントである`UserCard`にPropsを送信することができました！

それでは`app.vue`の全体のソースコードを書いておきますね。
```vue: app.vue
<script setup lang="ts">
interface Member {
  name: string;
  id: number;
  email: string;
  note?: string;
}

const memberListInit = new Map<number, Member>();
memberListInit.set(1, {
  id: 1,
  name: '田中太郎',
  email: 'taro.tanaka@example.com',
  note: 'ノートです。'
});
memberListInit.set(2, {
  id: 2,
  name: '鈴木次郎',
  email: 'suzuki.ziro@example.com'
});
const memberList = ref(memberListInit);
</script>

<template>
  <div>
    <UserCard
      v-for="[id, member] in memberList"
      :id="id"
      :key="id"
      :name="member.name"
      :email="member.email"
      :note="member.note"
    />
  </div>
</template>
```

## Propsを受け取る

次に`UserCard.vue`でPropsを受け取りましょう。

こっちでも`app.vue`と同じようなinterfaceを用意しましょう。
```ts: components/UserCard.vue
interface Props {
  name: string;
  id: number;
  email: string;
  note?: string;
}
```
このとき、interfaceの中は`v-bind`で渡したときの**属性名**を指定してください。

そして、Propsを受け取りましょう。
受け取るには`defineProps<Props>()`を使います。
```ts: components/UserCard.vue
const props = defineProps<Props>();
```
この関数は、親コンポーネントで`v-bind`で送信されたPropsを受け取る関数です。

これでデータの受け取りができるようになりました！

次に受け取ったPropsを使っていきましょう。

といっても普通の変数のように埋め込めば大丈夫です。

```vue: components/UserCard.vue
<template>
  <div class="userCard">
    <h2> {{ props.name }} </h2>
    <p> ID: {{ props.id }} </p>
    <p> email: {{ props.email }} </p>
    <p> {{ props.note }} </p>
  </div>
</template>
```
これで表示されるのですが、`props.name`のように`props`を指定しなくても`name`だけでも表示することができるんです。
これは、defineProps関数を使うことによりこれができるようになるらしいです。

```vue: components/UserCard.vue
<template>
  <div class="userCard">
    <h2> {{ name }} </h2>
    <p> ID: {{ id }} </p>
    <p> email: {{ email }} </p>
    <p> {{ note }} </p>
  </div>
</template>
```

それでは最後に、Propsをスクリプトで制御してみましょう！

`note`はinterfaceでオプショナルになっているので、この値がないときもあります。
そこで、データがないときに代わりの文字を表示するようにしてみましょう。

そのとき、スクリプト内では`<template>`のように`props`を省略することは**できません**ので注意してください。

```ts: components/UserCard.vue
const localNote = computed<string>(() => {
  return props.note ?? '--';
});
```

こんな感じでよさそうですね。
ではこれを実装しましょう。
```vue: components/UserCard.vue
<template>
  <div class="userCard">
    <h2> {{ name }} </h2>
    <p> ID: {{ id }} </p>
    <p> email: {{ email }} </p>
    <p> {{ localNote }} </p>
  </div>
</template>
```

これで実装できました！

それでは最後に`components/UserCard.vue`のソースコードを書いておきますね。
```vue: components/UserCard.vue
<script setup lang="ts">
interface Props {
  name: string;
  id: number;
  email: string;
  note?: string;
}

const props = defineProps<Props>();

const localNote = computed<string>(() => {
  return props.note ?? '--';
});
</script>

<template>
  <div class="userCard">
    <h2> {{ name }} </h2>
    <p> ID: {{ id }} </p>
    <p> email: {{ email }} </p>
    <p> {{ localNote }} </p>
  </div>
</template>

<style scoped>
.userCard {
  border: 1px solid green;
}
</style>
```

# まとめ

今回は、`Props`を使った親コンポーネントから子コンポーネントへのデータの受け渡しについて解説しました。

- `Props`は親から子にデータを渡すための仕組み。
- 親コンポーネントでは`v-bind`または省略形の`:`を使ってデータを送信。
- 子コンポーネントでは`defineProps`を使ってデータを受け取る。

次回は、`Emit`や`State`を使ったデータのやり取りについて解説する予定です。

それではお疲れさまでしたー🍎