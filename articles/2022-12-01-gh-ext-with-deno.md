---
title: "Deno で GitHub CLI 拡張機能書く"
emoji: "🔌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [deno, github, githubcli, gh]
published: true
---

この記事は[Deno Advent Calendar 2022](https://qiita.com/advent-calendar/2022/deno) 1 日目の記事です。

今更感がすごいですが、GitHub の公式 CLI `gh` の拡張機能を Deno で書く方法をいくつか紹介します。

## インストール済みの Deno を使用する

GitHub CLI 拡張機能はシェルスクリプトで書くことができるため、インストール済みの Deno を使用する場合は単に `deno run` するだけで作成できます。

利点:
- 開発の手間がかからない

欠点
- Deno のインストール必須

```sh:gh-extension
#!/usr/bin/env bash
deno run -q "${SCRIPT_DIR}/main.ts" "$@"
```

---

[`gh deno`](https://github.com/yskszk63/gh-deno) という GitHub CLI 拡張機能を経由することで、Deno のインストール機能も付けることができます。
[`gh-traffic`](https://github.com/yskszk63/gh-traffic) という拡張機能ではこの手法が使われています。

https://github.com/yskszk63/gh-deno

```sh:gh-extension
#!/usr/bin/env bash
gh deno --ping > /dev/null 2>&1 || gh extension install yskszk63/gh-deno
exec gh deno run -q "${SCRIPT_DIR}/main.ts" "$@"
```

## Node.js でも動くように変換する

`dnt` で npm パッケージに変換できるので、変換後のソースをコミットすることで Node.js でも動かせるようになります。

利点:
- Deno が未インストールでも Node.js で動作可能

欠点
- 結局 Deno か Node.js のどちらかはインストール必須
- リポジトリに変換後のソースをコミットする必要がある

今回は Node.js で実行だけできれば十分なため、`cjs`, `.d.ts`, `.ts` 形式の出力を全てオフにし、`esm` のみ出力させています。
https://github.com/proudust/gh-describe/blob/d091cdd61363c2cb09ba1e53d0a1bc4c04ef5900/_build.ts#L17-L29

筆者が作成した [`gh-describe`](https://github.com/proudust/gh-describe) という拡張機能では、変換後のソースを `esbuild` でバンドルしてコミットしています。
https://github.com/proudust/gh-describe/blob/d091cdd61363c2cb09ba1e53d0a1bc4c04ef5900/_build.ts#L59-L65

後はシェルスクリプトで `deno` コマンドが使用可能かどうかで分岐
```sh:gh-extension
#!/usr/bin/env bash
if type deno >/dev/null 2>&1; then
    deno run -q "${SCRIPT_DIR}/main.ts" "$@"
else
    node "${SCRIPT_DIR}"/dist/main.js "$@"
fi
```

## `deno compile` でシングルバイナリ化した Deno を使用する (未確認)

事例はありませんが、公式ドキュメントの[`gh extension create` を使用して Go 以外のプリコンパイル済み拡張機能を作成する](https://docs.github.com/ja/github-cli/github-cli/creating-github-cli-extensions#creating-a-non-go-precompiled-extension-with-gh-extension-create)の手順で `script/build.sh` に `deno compile` を行うスクリプトを記述することで、Deno も Node.js もインストール不要な拡張機能を Deno で書くことができます。

利点:
- Deno が未インストールでも動作可能

欠点
- 容量が大きい (linux 向け `gh-describe` が約 100 MB)

## 参考

- [GitHub CLI 拡張機能の作成 - GitHub Docs](https://docs.github.com/ja/github-cli/github-cli/creating-github-cli-extensions)
