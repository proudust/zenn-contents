---
title: リポジトリの Contributions を表示する Repobeats がいい感じ
emoji: 📊
type: tech
topics: [github]
published: false
---

[dotnet/maui](https://github.com/dotnet/maui#stats) の README.md に見覚えのないカラフルな表示が出ていました。

![dotnet/maui の Repobeats 表示](https://repobeats.axiom.co/api/embed/f917a77cbbdeee19b87fa1f2f932895d1df18b71.svg)

どうやらこれは [Axiom](https://www.axiom.co) という会社が無料で提供している [Repobeats](https://repobeats.axiom.co) という Web サービスによって生成されている画像のようです。
過去 30 日間のリポジトリに対するコントリビュートに関する以下のような統計情報を表示しているようです。

- コントリビュート数と草
- 新しく作成された Issues とクローズされた Issues の比率
- 新しく作成された Pull Request 数
- コミット数
- 日ごとの Issues, Pull Request, Push & Commet 数の統計
- コントリビューターの内、コントリビュート数上位 5 名の草

開発が活発なリポジトリのアピールのために張るのはもちろん、自作ツールや競プロ用のリポジトリに張って振り返るのも良さそうです。

---

せっかくなので README.md に Repobeats を置いているリポジトリの統計も覗いてみましょう。

- [dotnet/Comet](https://github.com/dotnet/Comet) ![dotnet/Comet](https://repobeats.axiom.co/api/embed/f917a77cbbdeee19b87fa1f2f932895d1df18b56.svg)
- [vueuse/vueuse](https://github.com/vueuse/vueuse/) ![vueuse/vueuse](https://repobeats.axiom.co/api/embed/a406ba7461a6a087dbdb14d4395046c948d44c51.svg)
