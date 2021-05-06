---
title: "GitHub Actions で hub コマンドを使ってプルリクエストをつくる"
emoji: "🤖"
type: "tech"
topics: ["github", "githubactions"]
published: true
---

[GitHub Actions](https://github.co.jp/features/actions) で [hub](https://github.com/github/hub) コマンドを使って Pull Request を自動化したときのメモです。

## 切っ掛け

Git-flow でブランチの運用をしていると、 release ブランチから main (master) ブランチと develop ブランチの2つへのマージを行う必要あります。この「2つのブランチにマージする」を GitHub Actions で自動化できないかなと考えていました。

そのときに、たまたま GitHub のコマンドラインツールである [hub](https://github.com/github/hub) の README に以下の記載があるのを見つけました。

> hub is ready to be used in your GitHub Actions workflows

`hub` コマンドを使えば `hub pull-request` でプルリクエストを送れるはずなので、試してみました。

## 作ったもの

Node.js のプロジェクトで、 `release/v{バージョンの番号}` というブランチが新しく作られたら、 `package.json` にあるバージョンを書き換えてコミットし、 main ブランチと dev ブランチへプルリクエストを送るという workflow を作りました。

```yml:pullrequest.yml
on: create
jobs:
  release-pr:
    if: ${{ contains(github.ref, 'refs/heads/release/v') }}
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Extract version
        shell: bash
        run: |
          echo "::set-output name=version::${GITHUB_REF#refs/heads/release/v}"
        id: extract_version
      - name: Setup node
        uses: actions/setup-node@v2
        with:
          node-version: '14'
      - run: |
          git config --global user.name "GitHub Action"
          git config --global user.email "41898282+github-actions[bot]@users.noreply.github.com"
      - name: Update package version
        shell: bash
        run: |
          npm version ${{ steps.extract_version.outputs.version }} -m "Release %s"
      - name: Create pull request
        shell: bash
        run: |
          hub push
          hub pull-request -b main -m "Release ${{ steps.extract_version.outputs.version }}"
          hub pull-request -b develop -m "Release ${{ steps.extract_version.outputs.version }}"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

「特定の名前のブランチが作られたとき」という判定は以下を参考にしました。

https://github.community/t/trigger-job-on-branch-created/16878/5


## おわりに

GitHub Actions でプルリクエストを送る方法を調べると、今まで GitHub Actions を触ったことがない初心者にはよくわからない内容ばかり出てきて困っていましたが、比較的短いコードで実現できて助かりました。

さらに調べたところ、 [git-flow-merge-action](https://github.com/marketplace/actions/git-flow-merge-action) という Git-flow のマージを自動でやってくれるありがたい Action があるようでした。  
こちらを使う方が簡単そうな気もしたのですが、 GitHub Actions 初心者なので色々試してみたかったのと、 main ブランチと develop ブランチに直接コミットできないようプロテクトをかけていたので hub コマンドでプルリクエストを送るようにしました。

機会があればこちらの Action や、別の方法も試してみたいと思います。

お読みいただきありがとうございました。
