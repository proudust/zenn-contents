---
title: "Deno でカスタム GitHub アクション書く"
emoji: "🦾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [deno, github, githubactions]
published: false
---

この記事は[GitHub Actions Advent Calendar 2022](https://qiita.com/advent-calendar/2022/github-actions) 2 日目の記事です。

GitHub Actions 上で動作するカスタムアクションを Deno で書く方法をいくつか紹介します。

## 複合アクションで denoland/setup-deno を実行する

複合アクションで denoland/setup-deno 後に `deno run` するだけのシンプルな手法です。
[hasundue/denopendabot](https://github.com/hasundue/denopendabot) というカスタムアクションではこの手法が使われています。

利点:
- 開発の手間がかからない

欠点
- denoland/setup-deno の分だけ時間がかかる (と言っても1~3秒程度)

```yaml:action.yaml
runs:
  using: composite
  steps:
    - name: Setup Deno
      uses: denoland/setup-deno@v1
      with:
        deno-version: v1.x
    - name: Deno Run
      run: deno run -q -A main.ts
```

## `dnt` で変換して JavaScript アクションとして実行する

[Deno で GitHub CLI 拡張機能書く](https://zenn.dev/proudust/articles/2022-12-01-gh-ext-with-deno#Node.js%20%82%C5%82%E0%93%AE%82%AD%82%E6%82%A4%82%C9%95%CF%8A%B7%82%B7%82%E9)でも紹介した `dnt` を使用した手法です。
筆者が作成した [`gh-describe`](https://github.com/proudust/gh-describe) というカスタムアクションではこの手法を採用しています。

利点:
- denoland/setup-deno を実行しない分速い (と言っても1~3秒程度)

欠点
- リポジトリに変換後のソースをコミットする必要がある

今回も Node.js で実行だけできれば十分なため、`esm` のみ出力して `esbuild` でバンドルしてコミットしています。
https://github.com/proudust/gh-describe/blob/d091cdd61363c2cb09ba1e53d0a1bc4c04ef5900/_build.ts#L17-L29
https://github.com/proudust/gh-describe/blob/d091cdd61363c2cb09ba1e53d0a1bc4c04ef5900/_build.ts#L52-L58

## 参考

- [Creating a composite action - GitHub Docs](https://docs.github.com/ja/actions/creating-actions/creating-a-composite-action)
