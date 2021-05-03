---
title: "vue-sfc-rollup と GitHub Packages でコンポーネントを配布する"
emoji: "💣"
type: "tech"
topics: ["vue", "github"]
published: true
---

[vue-sfc-rollup](https://github.com/team-innovation/vue-sfc-rollup) と [GitHub Packages](https://github.co.jp/features/packages) で Vue のコンポーネントを配布して、他のプロジェクトで利用できるようにしたときのメモです。

## 切っ掛け

プライベートで関わっているコミュニティで、「こういうことブラウザでできたらいいよね」とかでWebサイトとかサービスとかを作ることがあるのですが、プログラムは書けてもデザインは分からないみたいな方が多く、機能としては足りてるけど見た目がちょっと…ということがありました。

プログラマーが見た目を整えることに時間をかけなくて済むように、また同じコミュニティのサービスなのに見た目が全然違うものになってしまうのを防ぐため、Vueのコンポーネントを作ろう！と勝手にぼっちで作りました。

## vue-sfc-rollup を使う

[vue-sfc-rollup](https://github.com/team-innovation/vue-sfc-rollup) という、Vue のコンポーネントを公開できるように準備されたリポジトリがあるので、以下記事を参考にプロジェクトをつくりました。

https://qiita.com/mitsuya_bauhaus/items/021ba98f849bff614ab0

https://dev.to/htech/creating-a-vue-module-with-rollup-and-typescript-4igb

### プロジェクト生成

vue-sfc-rollup の README に記載されているコマンドを実行してプロジェクトをつくります。あまりグローバルにごちゃごちゃと入れたくないので `npx` で実行しました。

```bash
$ npx vue-sfc-rollup

? Which version of Vue are you writing for?
> Vue 2
  Vue 3
# Vue 2 を選択

? Is this a single component or a library?
  Single Component
> Library
# 複数のコンポーネントを作りたいので Library を選択

? What is the npm name of your library?
test-library
# 公開するときのパッケージ名を入力

? Will this library be written in JavaScript or TypeScript?
> JavaScript
  TypeScript
# TypeScript は勉強しようと思いつつもおサボり中なので JavaScript を選択

? Enter a location to save the library files
./test-library
# フォルダをつくる場所を入力
```

コマンド実行後、 `cd` とかでプロジェクトのディレクトリに移動し、 `npm install` で依存関係をインストールします。インストール完了後 `npm run serve` を実行すると、ローカルサーバが起動し、いい感じにしてくれます。

### コンポーネントの追加

コンポーネントを追加するには、 `/src/lib-components` フォルダに `.vue` ファイルを追加します。

```vue:TestComponentA.vue
<template>
  <div class="test-component-a">
    えー
  </div>
</template>

<script>
export default {
  name: 'TestComponentA',
}
</script>
```

```vue:TestComponentB.vue
<template>
  <div class="test-component-b">
    びー
  </div>
</template>

<script>
export default {
  name: 'TestComponentB',
}
</script>
```

コンポーネント作成後、 `/src/lib-components` にある `index.js` に追記します。複数のコンポーネントを登録する場合は以下のように記述します。

```js:index.js
export { default as TestComponentA } from './TestComponentA.vue'
export { default as TestComponentB } from './TestComponentB.vue'
```

### 配布用ファイルの生成

コンポーネントをつくったらビルドします。実行すると、 `/dist` に配布用のファイルが生成されます。

```bash
$ npm run build
```

## GitHub Packages で配布する

[GitHub Packages](https://github.co.jp/features/packages) は GitHub が提供しているパッケージのホスティングサービスです。

コンポーネントのソースコードは GitHub で管理しています。パッケージの配布も GitHub で行ったほうが管理がしやすそうだったので、以下ドキュメントを参考に GitHub Packages で配布することにしました。

https://docs.github.com/ja/packages/quickstart

https://zenn.dev/odiak/articles/950f05a34e9c84

### package.json の変更

`package.json` の `name` に公開するパッケージの名前を設定します。このとき、パッケージを公開する GitHub アカウントのユーザ名を先頭につけたものを設定します。

```json:package.json
{
  "name": "@hiratake/test-library"
}
```

また、 [こちら](https://docs.github.com/ja/actions/guides/publishing-nodejs-packages#about-package-configuration) のドキュメントに以下記載があります。

> package.jsonファイルにpublishConfigフィールドを設定するステップをワークフローに追加したなら、setup-nodeアクションを使ってregistry-urlを指定する必要はありませんが、パッケージを公開するレジストリは1つだけに限られます。

今回は GitHub Packages でしか公開しないので、以下を参考に `package.json` に `publishConfig` フィールドを設定します。

https://docs.npmjs.com/cli/v7/configuring-npm/package-json#publishconfig

```json:package.json
{
  "publishConfig": {
    "registry": "https://npm.pkg.github.com/hiratake"
  }
}
```

### パッケージの公開

[先程のドキュメント](https://docs.github.com/ja/packages/quickstart) では GitHub Actions でパッケージを公開する手順について説明されていましたが、とりあえず CLI から手動で公開してみます。

```bash
$ npm login --registry=https://npm.pkg.github.com

# 公開前にビルド
$ npm run build

$ npm pack
$ npm publish
```

パッケージの公開が成功すると、 GitHub のリポジトリページに `Packages` が表示され、公開されていることが確認できます。  
`package.json` に `description` を設定しておくと、パッケージの一覧表示のときにわかりやすいです。

## おわりに

自前でコンポーネントを公開する環境を整えようとすると大変そうなので、簡単に準備ができるものがあり助かりました。つくってくださった方に感謝ですね。

作ったものが以下です。

https://github.com/jaoafa/jao-ui

私もデザインを考えるのが得意とか専門で勉強してきたとかではないので、作ってはみたものの使いやすいかと言われると微妙ですが、よくないところは今後改善だな～というお気持ち。あといい加減 TypeScript やるべき。

お読みいただきありがとうございました。
