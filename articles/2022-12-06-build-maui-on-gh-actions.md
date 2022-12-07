---
title: "MAUI アプリを GitHub Actions 上でビルドする"
emoji: "🏗️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [dotnet, maui, githubactions]
published: true
---

この記事は[C# Advent Calendar 2022](https://qiita.com/advent-calendar/2022/csharplang) 6 日目の記事です。
最近遅刻常習者になりつつあるので気を付けます……。

先日 MAUI を GitHub Actions 上でビルドしようとして地味に苦戦したので書いておきます。

## TFM とビルド (発行) 可能な OS

.NET 7.0.100 現在、各 OS でビルド (発行) 可能な TargetFramework は以下のようになっています。 (`net6.0` も同様のため省略)

| TargetFrameworks     | Linux | MacOS | Windows |
| -------------------- | :---: | :---: | :-----: |
| `net7.0-android`     | ✅[^1] |   ✅   |    ✅    |
| `net7.0-ios`         |       |   ✅   |  ⚠️[^2]  |
| `net7.0-maccatalyst` |       |   ✅   |  ⚠️[^2]  |
| `net7.0-windows`     |       |       |    ✅    |

このように見事にバラバラなため、それぞれの OS でビルド or 発行するジョブを書いていく必要があります。面倒ですね。
今回はビルドが通ることのみを確認するため、`net7.0-android` は `ubuntu-latest`、他は `windows-latest` でビルドしてみます。

[^1]: 要 `.csproj` ファイル修正

[^2]: ビルドのみ可能、発行はエラー

## Linux 上で Android 向けビルドができるように修正する

VS 2022 の .NET MAUI アプリテンプレートからプロジェクトを作成した場合、Linux 上でビルドを実行すると下記のようなエラーが発生します。

```
$ dotnet build MauiSandbox/MauiSandbox.Net7.csproj -c Debug -f net7.0-android
MSBuild version 17.4.0+18d5aef85 for .NET
  Determining projects to restore...
Error: /usr/share/dotnet/sdk/7.0.100/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.Sdk.ImportWorkloads.targets(38,5): error NETSDK1178: The project depends on the following workload packs that do not exist in any of the workloads available in this installation: Microsoft.ios.Sdk.net7 [/home/runner/work/MauiSandbox/MauiSandbox/MauiSandbox/MauiSandbox.Net7.csproj::TargetFramework=net7.0-ios]
Error: /usr/share/dotnet/sdk/7.0.100/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.Sdk.ImportWorkloads.targets(38,5): error NETSDK1178: You may need to build the project on another operating system or architecture, or update the .NET SDK. [/home/runner/work/MauiSandbox/MauiSandbox/MauiSandbox/MauiSandbox.Net7.csproj::TargetFramework=net7.0-ios]
```

何故か二回同じエラーが表示されますが、Linux 上で `net7.0-ios` `net7.0-maccatalyst` がビルドできないことに起因するエラーのようで、`-f` オプションを付けていても発生するようでした。
この問題は `csproj` を下記のように編集し、Linux 上では `net7.0-ios` `net7.0-maccatalyst` を TFMs から外すことで回避できます。

```diff xml
  <PropertyGroup>
-   <TargetFrameworks>net7.0-android;net7.0-ios;net7.0-maccatalyst</TargetFrameworks>
+   <TargetFrameworks>net7.0-android</TargetFrameworks>
+   <TargetFrameworks Condition="!$([MSBuild]::IsOSPlatform('linux'))">$(TargetFrameworks);net7.0-ios;net7.0-maccatalyst</TargetFrameworks>
  </PropertyGroup>
```

## matrix で並列ビルドしてみる

発行まで行わないのであればビルドまでのステップはどの TFM でも同一のため、matrix で並列ビルドさせてみましょう。
Android のみ `ubuntu-latest` でビルドする必要があるため別ジョブに切り出す必要があるかと思っていましたが、JavaScript のような論理積、論理和の悪用(?)をすることで一つのジョブにまとめることができます。

https://qiita.com/technote-space/items/cbeed6ddd0488499afaa

```yaml
    runs-on: ${{ (contains(matrix.tfm, 'android') && 'ubuntu-latest') || 'windows-latest' }}
    strategy:
      matrix:
        tfm: [net7.0-android, net7.0-ios, net7.0-maccatalyst, net7.0-windows10.0.19041.0]
```

`net7.0-ios`、`net7.0-maccatalyst` を `macos-12` [^3] でビルドさせたい場合は下記のようにすれば一応できましたが、あまりにも可読性が低いのでおすすめはしません。

```yaml
    runs-on: ${{ (contains(matrix.tfm, 'android') && 'ubuntu-latest') || (contains(matrix.tfm, 'windows') && 'windows-latest') || 'macos-12' }}
    strategy:
      matrix:
        tfm: [net7.0-android, net7.0-ios, net7.0-maccatalyst, net7.0-windows10.0.19041.0]
```

[^3]: 2022-12-07 現在、`macos-latest` は `macos-11` のエイリアスになっているため Xcode のバージョンが古く、ビルドがうまくいかないので気を付けましょう。(1敗)

## 発行までしたい場合

今回はビルドのみを行いましたが、発行まで行いたい場合は下記の記事の手順で行うことができます。

https://blog.taranissoftware.com/building-net-maui-apps-with-github-actions
## おまけ: Linux 上で Windows 向けビルドはできる？

先日 .NET 7 から Linux 上で Windows 向け WPF アプリがビルドできるようになったらしく、もしやと思って MAUI でも試してみました。

https://tech.guitarrapc.com/entry/2022/11/11/031555

.NET MAUI アプリテンプレートでは `net7.0-windows10.0.19041.0` が Windows でのみ TFMs に含まれる設定になっているため、これを MacOS 以外なら TFMs に含まれるよう変更します。
また Linux では `EnableWindowsTargeting` `RuntimeIdentifiers` の手動設定が必要なようなので追加しておきます。

```diff xml
  <PropertyGroup>
-   <TargetFrameworks Condition="$([MSBuild]::IsOSPlatform('windows'))">$(TargetFrameworks);net7.0-windows10.0.19041.0</TargetFrameworks>
+   <TargetFrameworks Condition="!$([MSBuild]::IsOSPlatform('macos'))">$(TargetFrameworks);net7.0-windows10.0.19041.0</TargetFrameworks>
+   <EnableWindowsTargeting Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'windows'">true</EnableWindowsTargeting>
+   <RuntimeIdentifiers Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'windows'">win-x64</RuntimeIdentifiers>
  </PropertyGroup>
```

これで `dotnet build` してみたところ、以下のようなエラーが発生しました。

```
$ dotnet build MauiSandbox/MauiSandbox.Net7.csproj -c Debug -f net7.0-windows10.0.19041.0
MSBuild version 17.4.0+18d5aef85 for .NET
  Determining projects to restore...
  Restored /home/runner/work/MauiSandbox/MauiSandbox/MauiSandbox/MauiSandbox.Net7.csproj (in 6.66 sec).
/home/runner/.nuget/packages/microsoft.windowsappsdk/1.1.5/buildTransitive/Microsoft.UI.Xaml.Markup.Compiler.interop.targets(559,9): error MSB3073: The command ""/home/runner/.nuget/packages/microsoft.windowsappsdk/1.1.5/buildTransitive/../tools/net5.0/../net472/XamlCompiler.exe" "../MauiSandbox.Net7/obj/Debug/net7.0-windows10.0.19041.0/input.json" "../MauiSandbox.Net7/obj/Debug/net7.0-windows10.0.19041.0/output.json"" exited with code 1. [/home/runner/work/MauiSandbox/MauiSandbox/MauiSandbox/MauiSandbox.Net7.csproj::TargetFramework=net7.0-windows10.0.19041.0]
```

ファイルパスから察するに、MAUI が依存している Win UI 3 が .NET Framework で動作する XAML コンパイラを使用してしまっているため、正常動作しないようです。
この分だとWPF と同じで忘れたころに Linux ビルドできるようになりそうですね。
