---
title: double.Parse("1.5") → FormatException ← は？
emoji: "🤔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [csharp]
published: true
---

## TL;DR

.NET の `Parse` や `TryParse` は PC の言語設定によって動作が変わるので、国外で実行する可能性がある場合は第 2 引数を指定しよう。

## 再現コード

以下のコードで再現できます。

``` cs
using System.Globalization;
using System.Threading;

Thread.CurrentThread.CurrentCulture = CultureInfo.CreateSpecificCulture("fr-FR");
double.Parse("1.5"); // FormatException
```

[.NET Fiddle で実行する](https://dotnetfiddle.net/0U0yj6)

## 何が起きてるか

日本では小数点を表す記号として `.` (ピリオド) が使われますが、国によっては `,` (カンマ) や `٫` (Momayyez) を使って表す場合があります。[^1]  
.NET はこれを考慮し、言語設定によって `Parse` などの挙動が変わるようになっています。  

再現コードではスレッドの言語設定をフランスに変更しています。  
フランスでは小数点に `,` を、整数の 3 桁ごとの区切りに `.` を用いるのが一般的だそうで、
整数の 3 桁ごとの区切りが中途半端なところにあるために例外が起きているわけです。  

[^1]: [小数点 - Wikipedia](https://ja.wikipedia.org/wiki/%E5%B0%8F%E6%95%B0%E7%82%B9)

## 解決方法

`Parse` や `TryParse` の第 2 引数に `IFormatProvider` を渡すと、言語設定によらず引数の言語でパースします。  
ここに `CultureInfo.InvariantCulture` を指定することで、PC の言語設定に依らず、
小数点を `.`、整数の 3 桁ごとの区切りに `,` を用いるようになります。  

``` cs
using System.Globalization;
using System.Threading;

Thread.CurrentThread.CurrentCulture = CultureInfo.CreateSpecificCulture("fr-FR");
double.Parse("1.5", CultureInfo.InvariantCulture); // 1.5
```

## 防止策

Visual Studio のコード分析の警告に [CA1305](https://docs.microsoft.com/ja-jp/visualstudio/code-quality/ca1305) があるので、コード分析の警告を見るようにすれば未然に防げそう……と思ったのですが、私の環境では警告を出してくれなかったので、何かしら設定が必要なのかも？  

現状はラッパーを用意してそっちを使うようにしたり、検出してくれるサードパーティ製のアナライザーを探して導入したりするしかなさそうです。
