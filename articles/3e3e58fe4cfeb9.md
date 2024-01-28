---
title: "Windowsでlibavifをビルドする"
emoji: "🖼️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["avif"]
published: true
---

## 環境構築

CMake、Ninja、NASMの3つが必要なのでインストールします。  

```sh
winget install NASM.NASM Kitware.CMake
```

Ninjaは[Github](https://github.com/ninja-build/ninja/releases/tag/v1.11.1)から`ninja-win.zip`を選択してダウンロード後展開し、適当なフォルダに`ninja.exe`を置いてパスを通します。[^1]  

[^1]: 一応`winget install Ninja-build.Ninja`が使えるはずですが、筆者の環境では失敗したので直接入れる方法をとっています。  

```sh
nasm --version
clang --version
ninja --version
```

と入力してバージョン情報が出力されればOKです。

## ビルド

基本は[GithubのREADME](https://github.com/AOMediaCodec/libavif?tab=readme-ov-file#build-everything-from-scratch)通りにコマンドを実行すればOKです。  
ですが執筆時点では`.\ext\libwebp\build\libsharpyuv.lib`として生成されるべきファイルが`.\ext\libwebp\build\sharpyuv.lib`として生成されてしまいその後のCMakeに失敗するので、renコマンドで名前を変更しています。  
それ以外のコマンドはREADMEに書いてあるものと一緒です。  

```sh
git clone -b v1.0.3 https://github.com/AOMediaCodec/libavif.git
cd libavif/ext
./aom.cmd
./libyuv.cmd
./libsharpyuv.cmd
./libjpeg.cmd
./zlibpng.cmd
cd ..
ren .\ext\libwebp\build\sharpyuv.lib libsharpyuv.lib
cmake -S . -B build -DBUILD_SHARED_LIBS=OFF -DAVIF_CODEC_AOM=ON -DAVIF_LOCAL_AOM=ON -DAVIF_LOCAL_LIBYUV=ON -DAVIF_LOCAL_LIBSHARPYUV=ON -DAVIF_LOCAL_JPEG=ON -DAVIF_LOCAL_ZLIBPNG=ON -DAVIF_BUILD_APPS=ON
cmake --build build --parallel
```

`.\ext\aom\build.libavif`以下に`aom.lib`が、`.\build\Release`以下に`avif.lib`が生成されます。  
また、以下のコマンドで`avifenc.exe`と`avifdec.exe`が生成できます。  

```sh
msbuild ".\build\libavif.sln" /m /verbosity:minimal /p:Configuration=Release
```
