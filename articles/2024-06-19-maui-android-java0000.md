---
title: "MAUI Android で `JAVA0000: Type ~ is defined multiple times` と怒られた"
emoji: "👯‍♀️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [dotnet, maui]
published: true
---

Visual Studio で MAUI アプリケーションを開発している場合、Android 向けビルドがエラーなしで失敗していることがあります。
その場合は大抵、出力タブにはちゃんとエラーメッセージが出力されているが、Visual Studio がちゃんと認識できていないだけのことが多いです。
そしてその場合は大抵以下のようなエラーメッセージであることが多いです。

```
java.exe error JAVA0000: Error in androidx.collection.collection-jvm.jar:androidx/collection/ArrayMapKt.class:
java.exe error JAVA0000: Type androidx.collection.ArrayMapKt is defined multiple times: %USERPROFILE%.nuget\packages\xamarin.androidx.collection.jvm\1.3.0.2\buildTransitive\net8.0-android34.0\..\..\jar\androidx.collection.collection-jvm.jar:androidx/collection/ArrayMapKt.class, %USERPROFILE%.nuget\packages\xamarin.androidx.collection.ktx\1.2.0.9\buildTransitive\net6.0-android31.0\..\..\jar\androidx.collection.collection-ktx.jar:androidx/collection/ArrayMapKt.class
java.exe error JAVA0000: Compilation failed
java.exe error JAVA0000: java.lang.RuntimeException: com.android.tools.r8.CompilationFailedException: Compilation failed to complete, origin: %USERPROFILE%.nuget\packages\xamarin.androidx.collection.jvm\1.3.0.2\buildTransitive\net8.0-android34.0\..\..\jar\androidx.collection.collection-jvm.jar
java.exe error JAVA0000: androidx/collection/ArrayMapKt.class
java.exe error JAVA0000: 	at com.android.tools.r8.utils.S0.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:135)
java.exe error JAVA0000: 	at com.android.tools.r8.D8.main(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:5)
java.exe error JAVA0000: Caused by: com.android.tools.r8.CompilationFailedException: Compilation failed to complete, origin: %USERPROFILE%.nuget\packages\xamarin.androidx.collection.jvm\1.3.0.2\buildTransitive\net8.0-android34.0\..\..\jar\androidx.collection.collection-jvm.jar:androidx/collection/ArrayMapKt.class
java.exe error JAVA0000: 	at Version.fakeStackEntry(Version_8.2.33.java:0)
java.exe error JAVA0000: 	at com.android.tools.r8.T.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:5)
java.exe error JAVA0000: 	at com.android.tools.r8.utils.S0.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:82)
java.exe error JAVA0000: 	at com.android.tools.r8.utils.S0.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:32)
java.exe error JAVA0000: 	at com.android.tools.r8.utils.S0.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:31)
java.exe error JAVA0000: 	at com.android.tools.r8.utils.S0.b(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:2)
java.exe error JAVA0000: 	at com.android.tools.r8.D8.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:42)
java.exe error JAVA0000: 	at com.android.tools.r8.D8.b(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:13)
java.exe error JAVA0000: 	at com.android.tools.r8.D8.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:40)
java.exe error JAVA0000: 	at com.android.tools.r8.utils.S0.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:122)
java.exe error JAVA0000: 	... 1 more
java.exe error JAVA0000: Caused by: com.android.tools.r8.utils.b: Type androidx.collection.ArrayMapKt is defined multiple times: %USERPROFILE%.nuget\packages\xamarin.androidx.collection.jvm\1.3.0.2\buildTransitive\net8.0-android34.0\..\..\jar\androidx.collection.collection-jvm.jar:androidx/collection/ArrayMapKt.class, %USERPROFILE%.nuget\packages\xamarin.androidx.collection.ktx\1.2.0.9\buildTransitive\net6.0-android31.0\..\..\jar\androidx.collection.collection-ktx.jar:androidx/collection/ArrayMapKt.class
java.exe error JAVA0000: 	at com.android.tools.r8.utils.Q2.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:21)
java.exe error JAVA0000: 	at com.android.tools.r8.utils.D2.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:54)
java.exe error JAVA0000: 	at com.android.tools.r8.utils.D2.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:10)
java.exe error JAVA0000: 	at java.base/java.util.concurrent.ConcurrentHashMap.merge(ConcurrentHashMap.java:2056)
java.exe error JAVA0000: 	at com.android.tools.r8.utils.D2.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:6)
java.exe error JAVA0000: 	at com.android.tools.r8.graph.m4$a.d(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:6)
java.exe error JAVA0000: 	at com.android.tools.r8.dex.c.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:61)
java.exe error JAVA0000: 	at com.android.tools.r8.dex.c.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:12)
java.exe error JAVA0000: 	at com.android.tools.r8.dex.c.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:9)
java.exe error JAVA0000: 	at com.android.tools.r8.D8.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:45)
java.exe error JAVA0000: 	at com.android.tools.r8.D8.d(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:17)
java.exe error JAVA0000: 	at com.android.tools.r8.D8.c(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:69)
java.exe error JAVA0000: 	at com.android.tools.r8.utils.S0.a(R8_8.2.33_429c93fd24a535127db6f4e2628eb18f2f978e02f99f55740728d6b22bef16dd:28)
java.exe error JAVA0000: 	... 6 more
java.exe error JAVA0000:
```

> ```
> Type androidx.collection.ArrayMapKt is defined multiple times: %USERPROFILE%.nuget\packages\xamarin.androidx.collection.jvm\1.3.0.2\buildTransitive\net8.0-android34.0\..\..\jar\androidx.collection.collection-jvm.jar:androidx/collection/ArrayMapKt.class, %USERPROFILE%.nuget\packages\xamarin.androidx.collection.ktx\1.2.0.9\buildTransitive\net6.0-android31.0\..\..\jar\androidx.collection.collection-ktx.jar:androidx/collection/ArrayMapKt.class
> ```

この例では `androidx.collection.ArrayMapKt` が複数定義されていることが原因でビルドに失敗しているようです。
また同じ `xamarin.androidx.collection.jvm` のバージョン違い、1.3.0.2 と 1.2.0.9 の二つを両方とも含めてビルドしようとしてしまったことが原因っぽいことがわかります。

---

この問題は nuget パッケージの依存関係が原因であることが多いので、どの nuget パッケージがバージョンいくつでインストールされているかを確認します。
コマンドプロンプトなどで以下のコマンドを実行することで確認できます。

```
dotnet list <csproj> package --framework <target-framework> --include-transitive
```

`<csproj>` には対象のプロジェクトファイルを、`<target-framework>` には `net8.0-android34.0` などのターゲットフレームワークを指定します。
`<target-framework>` は `net8.0-android` のような省略形が使えないので注意しましょう。

実行すると以下のような結果が得られます。

```
> dotnet list Project1.csproj package --framework net8.0-android34.0 --include-transitive
Project 'Project1' has the following package references
   [net8.0-android34.0]:
   Top-level Package                               Requested   Resolved
(略)

   Transitive Package                                              Resolved
(略)
   > Xamarin.AndroidX.Collection                                   1.3.0.2
   > Xamarin.AndroidX.Collection.Jvm                               1.3.0.2
   > Xamarin.AndroidX.Collection.Ktx                               1.2.0.9
(略)
```

出力がかなり長いので省略しましたが、`Xamarin.AndroidX.Collection` から始まるパッケージの内、`Xamarin.AndroidX.Collection.Ktx` だけバージョンが古いですね。
この場合はパッケージに `Xamarin.AndroidX.Collection.Ktx` の `1.3.0.2` をインストールしてあげることで直ります。
