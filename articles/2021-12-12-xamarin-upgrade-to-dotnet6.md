---
title: Xamarin.Forms プロジェクトを .NET6 へアップグレードする
emoji: 6️⃣
type: tech
topics: [csharp, dotnet, xamarin, xamarinforms]
published: true
---

:::message alert
2022/11 現在この記事の内容は陳腐化しており、更新中です。
記載されている手順通りにアップグレードを実施しても **iOS 向けビルドができない**ことを確認しているため、注意してください。
:::

:::message alert
Xamarin.Forms が .NET 6 に対応しないことが名言されました。
そのためこの記事で紹介される方法は、**公式にはサポートされません**。
特殊な事情が無い限り、**Xamarin.Forms プロジェクトを MAUI 移行せず .NET6 にアップグレードすることは避けてください**。

> we're not going to do a .NET 6 version of Xamarin.Forms.
>
> https://github.com/xamarin/Xamarin.Forms/issues/14829#issuecomment-1004142873
:::

:::message
[.NET 6 での破壊的変更](https://github.com/xamarin/xamarin-macios/issues/13087)のため、Xamarin.iOS / Xamarin.Mac のライブラリを .NET 6 プロジェクトから参照することができなくなりました。
そのため Xamarin.iOS / Xamarin.Mac のプロジェクトを .NET 6 移行する場合、ライブラリ側の対応も必要になるため注意してください。

> **Why did we not make net6.0-ios compatible with the existing Xamarin TFMs?**
> `net6.0-ios` (and the other Apple TFMs, `net6.0-tvos` and `net6.0-macos`) contains [breaking changes](https://github.com/xamarin/xamarin-macios/issues/13087) that makes binaries built for existing Xamarin TFMs unusable.
>
> https://github.com/dotnet/designs/blob/main/accepted/2021/net6.0-tfms/net6.0-tfms.md#why-did-we-not-make-net60-ios-compatible-with-the-existing-xamarin-tfms
:::

この記事は[祝 .NET 6 GA！(中略) Advent Calendar 2021](https://qiita.com/advent-calendar/2021/microsoft) 8 日目の記事です。
まあこの書き出しを書いている時点で既に 11 日なんですけどね……

---

日本時間 2021/11/08 に .NET 6 と Visual Studio 2022 がリリースされましたが、一方で Xamarin の余命宣告がされました。
サポート期限は ~~2 年後の 2023/11 まで~~ 2024/5/1 まで[^1]、それまでにまだリリースもされていない後継フレームワーク [MAUI](https://docs.microsoft.com/ja-jp/dotnet/maui/what-is-maui) への移行を済ませる必要があります。
https://devblogs.microsoft.com/xamarin/whats-new-in-xamarin-and-visual-studio-2022/

MAUI は .NET 6 以降のみのサポートとなる[^2]ため、従来の Xamarin.Forms から MAUI へ移行する場合、実行環境とフレームワークを同時に更新することとなり、両者の破壊的変更に悩まされることが予想されます。
この記事では MAUI への移行に備え、フレームワークを Xamarin.Forms のまま実行環境のみを .NET 6 に移行する手順を紹介します。

[^1]: https://dotnet.microsoft.com/en-us/platform/support/policy/xamarin
[^2]: https://github.com/dotnet/maui/wiki/Xamarin.Forms-vs-.NET-MAUI

----

## .NET 6 に移行すると何が変わる？

実際に移行する前に、.NET 6 に移行することで何が変わるのか確認しましょう。

|                                           | 従来の Xamarin.Forms      | .NET 6 の Xamarin.Forms | MAUI       |
| ----------------------------------------- | ------------------------- | ----------------------- | ---------- |
| フレームワーク                            | Xamarin.Forms             | Xamarin.Forms           | ✅ **MAUI** |
| フレームワークのサポート期限              | 2024/5/1                  | 2024/5/1                | ✅ 未発表   |
| 実行環境                                  | Xamarin (Mono)            | ✅ **.NET 6**            | .NET 6     |
| 実行環境のサポート期限                    | 2024/5/1                  | ✅ **2024/11/08**        | 2024/11/08 |
| `.csproj` の形式                          | Non-SDK-style / SDK-style | ⚠️ **SDK-style**         | SDK-style  |
| 既定の C# バージョン                      | C# 8.0                    | ✅ **C# 10.0**           | C# 10.0    |
| サポートされる Visual Studio のバージョン | 2019 / 2022               | ⚠️ **2022**              | 2022       |

🟦 フレームワークは Xamarin.Forms のまま

.NET 6 に移行したからといって Xamarin.Forms 側で特に変わることはありません。
サポート期限も変わらず 2024/5/1 です。
ちなみに次の LTS バージョンである .NET 8 のリリース予定は 2023/11 です。

✅ 実行環境が Mono から .NET 6 に

これにより、従来の `netstandard2.1` やプラットフォーム固有のライブラリに加え、`netcoreapp` `net5.0` `net6.0` のライブラリを使用できます。
.NET 6 までの性能向上の恩恵を受けられる半面、細かい破壊的変更が多数含まれているため注意が必要です。

筆者が特に注意が必要と考えている破壊的変更は以下の 2 つです。
アプリケーションコードで使用していない場合でもサードパーティー製ライブラリで利用している可能性もあるため気を付けましょう。

- [DeflateStream、GZipStream、CryptoStream での部分的な読み取りとゼロバイトの読み取り](https://docs.microsoft.com/ja-jp/dotnet/core/compatibility/core-libraries/6.0/partial-byte-reads-in-streams)
- [System.Drawing.Common が Windows でしかサポートされない](https://docs.microsoft.com/ja-jp/dotnet/core/compatibility/core-libraries/6.0/system-drawing-common-windows-only)

https://docs.microsoft.com/ja-jp/dotnet/core/compatibility/breaking-changes

⚠️ `.csproj` は SDK-style のみサポート

Visual Studio 2017 以降 `.csproj` に SDK-style と呼ばれる新しい書き方が増え、これまで冗長で人の読めるものではなかった `.csproj` が整理されました。
この書き方は .NET Standard や .NET Core のプロジェクトではデフォルトで反映され、.NET Framework でも自分で書き換えれば利用できましたが、Xamarin の各プラットフォーム向けプロジェクトでは利用できませんでした。
.NET 6 へ移行する場合は SDK-style への移行が必須になります。
と言ってもほとんど自動で直してくれるので、手で直す箇所はほんの数行です。

https://ufcpp.net/blog/2017/5/newcsproj/

✅ 既定の C# バージョンが 8.0 から 10.0 に

従来の Xamarin.Forms でも手順を踏めば C# 10.0 の機能を利用できましたが、その手順が少々面倒であったり、ランタイム側の修正を伴う[クラスの共変戻り値](https://ufcpp.net/study/csharp/cheatsheet/ap_ver9/#class-covariant-returns)が利用できなかったりしていました。
.NET 6 ではデフォルトで C# 10.0 の機能が全て利用できます。
[レコード型](https://ufcpp.net/study/csharp/cheatsheet/ap_ver9/#record)や [`not` パターン](https://ufcpp.net/study/csharp/datatype/patterns/?p=3#not-pattern)、[ファイル スコープ名前空間](https://ufcpp.net/study/csharp/cheatsheet/ap_ver10/#file-scoped-namespace)など便利な機能がたくさん増えているので、積極的に使っていきましょう。

https://ufcpp.net/study/csharp/cheatsheet/ap_ver9/
https://ufcpp.net/study/csharp/cheatsheet/ap_ver10/

⚠️ Visual Studio 2019 でのビルドが不可能に

Visual Studio 2019 では .NET 6 アプリケーションのビルドはサポートされていないので、.NET 6 に移行すると当然ビルドができなくなってしまいます。
しかも厄介なことに 2019 でビルドしようとすると `bin` `obj` フォルダが壊れてしまうようで、壊れてしまったこれらのフォルダを削除しないと 2022 でもビルドが通らなくなります。
気を付けましょう。（1 敗）

----

## Xamarin.Forms プロジェクトを .NET 6 へ移行する

違いがわかったところで .NET 6 への移行を始めましょう。

### 0. 前提条件

下記の手順は Visual Studio 2022 v17.4.1 及び .NET v6.0.202 で動作確認をしています。
Visual Studio for Mac での動作確認はしていないため、注意してください。
https://visualstudio.microsoft.com/

また実際に移行した際のソースコードは [github.com/proudust/XamarinSandbox](https://github.com/proudust/XamarinSandbox/tree/xamarin-update/net6.0) にあります。

:::message alert
[Xamarin.CommunityToolkit](https://www.nuget.org/packages/Xamarin.CommunityToolkit/) など下記条件を満たす NuGet パッケージを参照している場合、下記手順では正しく移行することができません。

- .NET 6 用バイナリが同梱されていない
- .NET Core 3.1 用バイナリが Windows 依存になっている
:::

### 1. `dotnet workload` コマンドで Android / iOS ワークロードをインストール

MAUI リリース前のバージョンの Visual Studio 2022 を使用する場合、.NET 6 の Android / iOS サポートを別途インストールする必要があります。
以下のコマンドでインストールできます。

```text
dotnet workload install android
dotnet workload install ios
```

### 2. `.csproj` を SDK-style に移行する

各プラットフォーム固有のプロジェクトの `.csproj` を SDK-style のような形式に移行して回ります。
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

:::message
共有プロジェクトは初めから SDK-style になっているはずなので、この手順では無視して問題ありません。
念のため確認したい場合は `.csproj` を開き、`Project` 要素に `Sdk` 属性がついているかを確認します。
Non-SDK-style だった場合はプラットフォーム固有のプロジェクト同様に `upgrade-assistant upgrade` コマンドを実行します。

```xml:SDK-style の例
<Project Sdk="Microsoft.NET.Sdk">
```

```xml:Non-SDK-style の例
<Project DefaultTargets="Build" ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
```
:::

### 3. `.csproj` を手作業で修正

下記のように変更します。

**共有プロジェクト**
- `TargetFramework` を `net6.0` に変更
- ([Xamarin.Forms.Visual.Material](https://www.nuget.org/packages/Xamarin.Forms.Visual.Material/) を参照している場合のみ) `XFDisableTargetFrameworkValidation` を `True` で追加

```diff xml:XamarinSandbox.csproj
   <PropertyGroup>
-    <TargetFramework>netstandard2.0</TargetFramework>
+    <TargetFramework>net6.0</TargetFramework>
+    <XFDisableTargetFrameworkValidation>True</XFDisableTargetFrameworkValidation>
   </PropertyGroup>
```

**Android プロジェクト**
- `TargetFramework` を `net6.0-android` に変更
- `PropertyGroup` に `<GenerateAssemblyInfo>false</GenerateAssemblyInfo>` を追加
- `RootNamespace` をアップグレード前の値で追加
- (Xamarin.Forms.Visual.Material を参照している場合のみ) `XFDisableTargetFrameworkValidation` を `True` で追加

```diff xml:XamarinSandbox.Android.csproj
   <PropertyGroup>
-    <TargetFramework>net6.0</TargetFramework>
+    <TargetFramework>net6.0-android</TargetFramework>
+    <GenerateAssemblyInfo>false</GenerateAssemblyInfo>
+    <RootNamespace>XamarinSandbox.Droid</RootNamespace>
+    <XFDisableTargetFrameworkValidation>True</XFDisableTargetFrameworkValidation>
   </PropertyGroup>
```

**iOS プロジェクト**
- `TargetFramework` を `net6.0-ios` に変更
- `PropertyGroup` に `<GenerateAssemblyInfo>false</GenerateAssemblyInfo>` を追加
- (Xamarin.Forms.Visual.Material を参照している場合のみ) `XFDisableTargetFrameworkValidation` を `True` で追加

```diff xml:XamarinSandbox.iOS.csproj
   <PropertyGroup>
-    <TargetFramework>net6.0</TargetFramework>
+    <TargetFramework>net6.0-ios</TargetFramework>
+    <GenerateAssemblyInfo>false</GenerateAssemblyInfo>
+    <XFDisableTargetFrameworkValidation>True</XFDisableTargetFrameworkValidation>
   </PropertyGroup>
```

:::message
`GenerateAssemblyInfo` はアセンブリ属性を自動生成する設定で、SDK-style ではデフォルトで true に設定されています。
Android / iOS プロジェクトの場合はテンプレートに存在する `Properties\AssemblyInfo.cs` の定義と重複してしまいコンパイルエラーとなってしまうため、明示的に無効にする設定を加えています。
今回は解説を省きますが、自動生成される属性を `Properties\AssemblyInfo.cs` から削除することでもコンパイルエラーを回避することができます。
:::

:::message
`RootNamespace` はプロジェクトの既定の名前空間を変更する設定で、SDK-style ではデフォルトでプロジェクト名に設定されています。
Android プロジェクトのみ `Xamarin.Forms.Android` 名前空間との衝突を避けるため、プロジェクト.Droid に変更する必要があります。
:::

:::message
Xamarin.Forms.Visual.Material には Xamarin.(Android|iOS) のバージョンチェック処理が含まれていますが、.NET 6 を使用している場合 Xamarin.(Android|iOS) 6.0 と誤認識されてしまいコンパイルエラーになります。
`XFDisableTargetFrameworkValidation` を `True` に設定することでこのバージョンチェック処理をスキップすることができます。[^3]
:::

[^3]: ソース: https://github.com/xamarin/Xamarin.Forms/blob/8af4f19be0cfb75f6a1df70c4db94eb6937ea1ee/.nuspec/Xamarin.Forms.Visual.Material.targets#L2

### 4. Visual Studio でソリューションを開き、構成マネージャを開く

ここまで行った後、Visual Studio 2022 でソリューションを開くと、「現在のソリューションには、正しくない構成マッピングが含まれています。（後略）」という表示が出ます。
指示に従い構成マネージャを開き、そのまま何も変更せずに閉じ、すべて保存します。
それだけで `XamarinSandbox.sln` が修正されるので、これで完了です。

### Ex. 本当に .NET 6 になったのか確かめてみる

せっかくなので本当に .NET 6 が Android や iOS 上で動作しているのか確かめてみましょう。

従来の Xamarin では .NET の Linux 向け実装である Mono が使用されていました。
その Mono に含まれる `Mono.Runtime` クラスの `Runtime.GetDisplayName()` クラスをリフレクションで取得し、実行することで確かめてみましょう。
下記のようなコードで取得することができます。[^4]

[^4]: 参考: https://social.msdn.microsoft.com/Forums/en-US/5dd995d7-9692-4647-9c7a-107129823ea0/how-to-check-my-mono-version

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

というわけで本物の .NET 6 が動いているようです。
