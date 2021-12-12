---
title: Xamarin.Froms プロジェクトを .NET6 へアップグレードする
emoji: 6️⃣
type: tech
topics: [csharp, dotnet, xamarin, xamarinforms]
published: true
---

この記事は[祝 .NET 6 GA！(中略) Advent Calendar 2021](https://qiita.com/advent-calendar/2021/microsoft) 8 日目の記事です。
まあこの書き出しを書いている時点で既に 11 日なんですけどね……

---

日本時間 2021/11/08 に .NET 6 と Visual Studio 2022 がリリースされましたが、一方で Xamarin の余命宣告がされました。
サポート期限は 2 年後の 2023/11 まで、それまでにまだリリースもされていない MAUI への移行を済ませる必要があります。
https://devblogs.microsoft.com/xamarin/whats-new-in-xamarin-and-visual-studio-2022/

MAUI は .NET 6 以降のみのサポートとなる[^1]ため、従来の Xamarin.Froms から MAUI へ移行する場合、実行環境と UI フレームワークを同時に更新することとなり、両者の破壊的変更に悩まされることが予想されます。
この記事では MAUI への移行に備え、UI フレームワークを Xamarin.Froms のまま実行環境のみを .NET 6 に移行する手順を紹介します。

[^1]: https://github.com/dotnet/maui/wiki/Xamarin.Forms-vs-.NET-MAUI

----

## .NET 6 に移行すると何が変わる？

実際に移行する前に、.NET 6 に移行することで何が変わるのか確認しましょう。

|                                           | 従来の Xamarin.Froms      | .NET 6 の Xamarin.Froms | MAUI       |
| ----------------------------------------- | ------------------------- | ----------------------- | ---------- |
| UI フレームワーク                         | Xamarin.Froms             | Xamarin.Froms           | ✅ **MAUI** |
| UI フレームワークのサポート期限           | 2023/11                   | 2023/11                 | ✅ 未発表   |
| 実行環境                                  | Xamarin (Mono)            | ✅ **.NET 6**            | .NET 6     |
| 実行環境のサポート期限                    | 2023/11                   | ✅ **2024/11/08**        | 2024/11/08 |
| `.csproj` の形式                          | Non-SDK-style / SDK-style | ⚠️ **SDK-style**         | SDK-style  |
| 既定の C# バージョン                      | C# 8.0                    | ✅ **C# 10.0**           | C# 10.0    |
| サポートされる Visual Studio のバージョン | 2019 / 2022               | ⚠️ **2022**              | 2022       |

🟦 UI フレームワークは Xamarin.Froms のまま

.NET 6 に移行したからといって Xamarin.Froms 側で特に変わることはありません。
サポート期限も変わらず 2023/11 です。
ちなみに 2023/11 は .NET 8 LTS がリリースされる予定の月でもあります。

✅ 実行環境が Mono から .NET 6 に

これにより、従来の `netstandard2.1` やプラットフォーム固有のライブラリに加え、`netcoreapp` `net5.0` `net6.0` のライブラリを使用できます。
.NET 6 までの性能向上の恩恵を受けられる半面、細かい破壊的変更が多数含まれているため注意が必要です。
といっても実際踏みそうな破壊的変更はコンパイラ警告がいくつか増えたのと、[.NET 6 で `DeflateStream` `GZipStream` `CryptoStream` の挙動が変更された](https://docs.microsoft.com/ja-jp/dotnet/core/compatibility/core-libraries/6.0/partial-byte-reads-in-streams)ぐらいだとは思いますが。

https://docs.microsoft.com/ja-jp/dotnet/core/compatibility/breaking-changes

⚠️ `.csproj` は SDK-style のみサポート

Visual Studio 2017 以降 `.csproj` に SDK-style と呼ばれる新しい書き方が増え、これまで冗長で人の読めるものではなかった `.csproj` が整理されました。
この書き方は .NET Standard や .NET Core のプロジェクトではデフォルトで反映され、.NET Framework でも自分で書き換えれば利用できましたが、Xamarin の各プラットフォーム向けプロジェクトでは利用できませんでした。
.NET 6 へ移行する場合は SDK-style への移行が必須になります。
と言ってもほとんど自動で直してくれるので、手で直す箇所はほんの数行です。

https://ufcpp.net/blog/2017/5/newcsproj/

✅ 既定の C# バージョンが 8.0 から 10.0 に

従来の Xamarin.Froms でも手順を踏めば C# 10.0 の機能を利用できましたが、その手順が少々面倒であったり、ランタイム側の修正を伴う[クラスの共変戻り値](https://ufcpp.net/study/csharp/cheatsheet/ap_ver9/#class-covariant-returns)が利用できなかったりしていました。
.NET 6 ではデフォルトで C# 10.0 の機能が全て利用できます。
[レコード型](https://ufcpp.net/study/csharp/cheatsheet/ap_ver9/#record)や [`not` パターン](https://ufcpp.net/study/csharp/datatype/patterns/?p=3#not-pattern)、[ファイル スコープ名前空間](https://ufcpp.net/study/csharp/cheatsheet/ap_ver10/#file-scoped-namespace)など便利な機能がたくさん増えているので、積極的に使っていきましょう。

⚠️ Visual Studio 2019 でのビルドが不可能に

Visual Studio 2019 では .NET 6 アプリケーションのビルドはサポートされていないので、.NET 6 に移行すると当然ビルドができなくなってしまいます。
しかも厄介なことに 2019 でビルドしようとすると `bin` `obj` フォルダが壊れてしまうようで、壊れてしまったこれらのフォルダを削除しないと 2022 でもビルドが通らなくなります。
気を付けましょう。（1 敗）


## Xamarin.Froms プロジェクトを .NET 6 へ移行する

違いがわかったところで .NET 6 への移行を始めましょう。
下記の手順は Visual Studio 2022 の Xamarin.Froms テンプレートで生成したソリューション `XamarinSandbox` で試したものになります。

### 1. `.csproj` を SDK-style に移行する

各プロジェクトの `.csproj` を SDK-style のような形式に移行して回ります。
.NET アップグレード アシスタントで移行してしまうのが早いです。
https://docs.microsoft.com/ja-jp/dotnet/core/porting/upgrade-assistant-overview

```text:cmd.exe
$ dotnet tool install -g upgrade-assistant
$ rd /s /q XamarinSandbox\XamarinSandbox.Android\bin XamarinSandbox\XamarinSandbox.Android\obj
$ upgrade-assistant upgrade XamarinSandbox\XamarinSandbox.Android\XamarinSandbox.Android.csproj
$ rd /s /q XamarinSandbox\XamarinSandbox.iOS\bin XamarinSandbox\XamarinSandbox.iOS\obj
$ upgrade-assistant upgrade XamarinSandbox\XamarinSandbox.iOS\XamarinSandbox.iOS.csproj
```

```text:sh
$ dotnet tool install -g upgrade-assistant
$ rmdir -r XamarinSandbox/XamarinSandbox.Android/bin XamarinSandbox/XamarinSandbox.Android/obj
$ upgrade-assistant upgrade XamarinSandbox/XamarinSandbox.Android/XamarinSandbox.Android.csproj
$ rmdir -r XamarinSandbox/XamarinSandbox.iOS/bin XamarinSandbox/XamarinSandbox.iOS/obj
$ upgrade-assistant upgrade XamarinSandbox/XamarinSandbox.iOS/XamarinSandbox.iOS.csproj
```

:::message
なぜか `bin` `obj` フォルダを事前に削除しておかないとエラーになるため、`rd` コマンドで削除しています。
:::

### 2. `.csproj` を手作業で修正

下記のように変更します。

- `TargetFramework` を `net6.0-android` または `net6.0-ios` に変更
- `PropertyGroup` に `<GenerateAssemblyInfo>false</GenerateAssemblyInfo>` を追加
- (Android のみ) `RootNamespace` をアップグレード前の値で追加

Android プロジェクトの場合以下のようになります。

```diff xml:XamarinSandbox.Android.csproj
   <PropertyGroup>
-    <TargetFramework>net6.0</TargetFramework>
+    <TargetFramework>net6.0-android</TargetFramework>
+    <GenerateAssemblyInfo>false</GenerateAssemblyInfo>
+    <RootNamespace>XamarinSandbox.Droid</RootNamespace>
   </PropertyGroup>
```

### 3. Visual Studio でソリューションを開き、構成マネージャを開く

ここまで行った後、Visual Studio 2022 でソリューションを開くと、「現在のソリューションには、正しくない構成マッピングが含まれています。（後略）」という表示が出ます。
指示に従い構成マネージャを開き、そのまま何も変更せずに閉じ、すべて保存します。
それだけで `XamarinSandbox.sln` が修正されるので、これで完了です。

### 4. Xamarin.Froms プロジェクトのターゲットフレームワークを .NET 6 に変更

ここまで来ると Visual Studio 上から Xamarin.Froms プロジェクトのターゲットフレームワークを .NET 6 に変更できます。
変更後、デバッグ実行して正常動作することを確認しましょう。


## 本当に .NET 6 になったのか確かめてみる

せっかくなので本当に .NET 6 が Android などの端末で動作しているのか確かめてみましょう。

従来の Xamarin では .NET の Linux 向け実装である Mono が使用されていました。
その Mono に含まれる `Mono.Runtime` クラスの `Runtime.GetDisplayName()` クラスをリフレクションで所得し、実行することで確かめてみましょう。
下記のようなコードで取得することができます。[^2]

[^2]: 参考: https://social.msdn.microsoft.com/Forums/en-US/5dd995d7-9692-4647-9c7a-107129823ea0/how-to-check-my-mono-version

```cs
string monoVersion = Type.GetType("Mono.Runtime")
    ?.GetMethods()
    .FirstOrDefault(x => x.Name == "GetDisplayName")
    ?.Invoke(null, null)
    ?.ToString();
string dotnetVersion = Environment.Version.ToString();
```

移行前後で実行してみると以下のような情報が取得できます。

|               | 移行前                         | 移行後  |
| ------------- | ------------------------------ | ------- |
| monoVersion   | `6.12.0 (2020-02/c633fe92383)` | (null)  |
| dotnetVersion | `4.0.50524.0`                  | `6.0.0` |
|               |                                |         |

というわけで本物の .NET 6 が動いているようです。
