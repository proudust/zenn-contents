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

## TFM とビルドまたは発行可能な OS

.NET 7.0.100 現在、各 OS でビルド (`dotnet build`) または発行 (`dotnet publish`) 可能な TargetFramework (以下 TFM) は以下の通りです。
表では省略していますが、`net6.0` でも同様です。

| TFM                  | Linux | MacOS | Windows |
| -------------------- | :---: | :---: | :-----: |
| `net7.0-android`     | ✅[^1] |   ✅   |    ✅    |
| `net7.0-ios`         |       |   ✅   |  ⚠️[^2]  |
| `net7.0-maccatalyst` |       |   ✅   |  ⚠️[^2]  |
| `net7.0-windows`     |       |       |    ✅    |

このように見事にバラバラなため、それぞれの OS でビルドまたは発行するジョブを書いていく必要があります。面倒ですね。
ビルドだけであれば Windows で全てビルド可能ですが、private リポジトリで実行することも考慮すると可能な限り Linux で済ませたいところです。

今回は発行まではせず、`net7.0-android` は Linux、他は Windows でビルドだけしてみます。

[^1]: 要 `.csproj` ファイル修正

[^2]: ビルドのみ可能、発行はエラー

## Linux 上で Android 向けビルドができるように修正

VS 2022 の .NET MAUI アプリテンプレートからプロジェクトを作成した場合、Linux 上でビルドを実行すると下記のようなエラーが発生します。

```
$ dotnet build MauiSandbox/MauiSandbox.Net7.csproj -c Debug -f net7.0-android
MSBuild version 17.4.0+18d5aef85 for .NET
  Determining projects to restore...
Error: /usr/share/dotnet/sdk/7.0.100/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.Sdk.ImportWorkloads.targets(38,5): error NETSDK1178: The project depends on the following workload packs that do not exist in any of the workloads available in this installation: Microsoft.ios.Sdk.net7 [/home/runner/work/MauiSandbox/MauiSandbox/MauiSandbox/MauiSandbox.Net7.csproj::TargetFramework=net7.0-ios]
Error: /usr/share/dotnet/sdk/7.0.100/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.Sdk.ImportWorkloads.targets(38,5): error NETSDK1178: You may need to build the project on another operating system or architecture, or update the .NET SDK. [/home/runner/work/MauiSandbox/MauiSandbox/MauiSandbox/MauiSandbox.Net7.csproj::TargetFramework=net7.0-ios]
```

何故か二回同じエラーが表示されますが、Linux 上で `net7.0-ios` `net7.0-maccatalyst` がビルドできないことに起因するエラーのようです。
TFMs に存在するだけでダメらしく、`-f` オプションを付けていても発生しました。

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
Android のみ Linux でビルドしたいため別ジョブに切り出す必要があるかと思いましたが、JavaScript のような論理積、論理和の悪用 (?) をすることで一つのジョブにまとめることができます。

https://qiita.com/technote-space/items/cbeed6ddd0488499afaa

```yaml
    runs-on: ${{ (contains(matrix.tfm, 'android') && 'ubuntu-latest') || 'windows-latest' }}
    strategy:
      matrix:
        tfm: [net7.0-android, net7.0-ios, net7.0-maccatalyst, net7.0-windows10.0.19041.0]
```

`net7.0-ios` `net7.0-maccatalyst` を MacOS [^3] でビルドさせたい場合は下記のようにすれば一応できましたが、あまりにも可読性が低いのでおすすめはしません。

```yaml
    runs-on: ${{ (contains(matrix.tfm, 'android') && 'ubuntu-latest') || (contains(matrix.tfm, 'windows') && 'windows-latest') || 'macos-12' }}
    strategy:
      matrix:
        tfm: [net7.0-android, net7.0-ios, net7.0-maccatalyst, net7.0-windows10.0.19041.0]
```

後は `dotnet workload restore` してから `dotnet build` するだけです。
GitHub Actions の GitHub-hosted runners では maui ワークフローがインストールされていないため、`dotnet workload restore` しないとエラーになります。
また、`dotnet build` する際も `-f` を付けないと全ての TFMs をビルドしてしまうため、重複してビルドしてしまいます。

```yaml
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: 7

      - name: Restore Workload
        run: dotnet workload restore

      - name: Build
        run: dotnet build MauiSandbox/MauiSandbox.Net7.csproj -c Release -f ${{ matrix.tfm }}
```

[^3]: 2022-12-07 現在、`macos-latest` は `macos-11` のエイリアスになっており、Xcode のバージョンが古いためビルドに成功しません。(1敗)
      MAUI アプリをビルドまたは発行する際は `macos-12` を使用しましょう。

## 発行までしたい場合

今回はビルドのみを行いましたが、発行まで行いたい場合は下記の記事の手順で行うことができます。

https://blog.taranissoftware.com/build-net-maui-apps-with-github-actions

## おまけ: Linux 上で Windows 向けビルドはできる？

.NET 7 から Linux 上で Windows 向け WPF アプリがビルドできるようになったそうです。

https://tech.guitarrapc.com/entry/2022/11/11/031555

これはもしやと思って MAUI でも試してみました。

.NET MAUI アプリテンプレートでは `net7.0-windows10.0.19041.0` が Windows でのみ TFMs に含まれる設定になっているため、これを MacOS 以外なら TFMs に含まれるよう変更します。
また Linux で Windows 向けビルドを行う場合は `EnableWindowsTargeting` `RuntimeIdentifiers` を明示的に指定する必要があるようなので追加しておきます。

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
