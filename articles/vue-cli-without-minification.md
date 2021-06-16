---
title: "Vue CLI の multi-page mode で HTML の minify を無効にする"
emoji: "🕌"
type: "tech"
topics: ["vue"]
published: true
---

[Vue CLI](https://cli.vuejs.org/) でつくった multi-page mode なプロジェクトで、縮小されていないHTMLをビルドできるようにしたときのメモです。

## 環境

- Vue CLI v4.5.13

## やったこと

minify 無効化は、調べると記事がいくつか見つかったのですが、 multi-page mode の場合だとエラーが出てビルドができませんでした。

https://qiita.com/nowri/items/96582106253e19514cb7

[公式のドキュメント](https://cli.vuejs.org/config/#pages) にある通り、 `pages` で指定したオブジェクトは直接 `html-webpack-plugin` に渡されます。  
なので、オブジェクトに minify を無効化するプロパティを追加すれば、縮小されていない状態のHTMLを書き出せました。

```js:vue.config.js
module.exports = {
  pages: {
    hoge: {
      entry: 'src/pages/hoge/main.js',
      template: 'public/hoge.html',
      filename: 'hoge.html',
      title: 'hoge',
      minify: false, // これを追加
    },
    fuga: {
      entry: 'src/pages/fuga/main.js',
      template: 'public/fuga.html',
      filename: 'fuga.html',
      title: 'fuga',
      minify: false, // これを追加
    },
  },
}
```

## おわりに

英語が得意でなく、公式ドキュメントをざっくりしか読めてなかったのでやたら時間がかかりました。イングリッシュぢからの向上を頑張らないといけないな～というお気持ち。

お読みいただきありがとうございました。
