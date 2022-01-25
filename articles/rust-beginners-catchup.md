---
title: "Rustを初め"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rust"]
published: false
---

# 10 秒で概要

- 当記事は、Rust を初めて最初の一歩として何から手を付けていいか分からない人向けの記事です。
  - 用語の整理
  - 環境のセットアップ
  - 写経に適したチュートリアルの紹介
  - 躓きポイントの紹介

**その他**

- 開発環境は、Windows10 を想定しています。
- VSCode は必須です。
- Docker は任意です。

# 用語の整理

当記事を読むうえで、押さえておくべき用語です。

- rustup

# 環境のセットアップ

- 当記事に記載の VSCode 環境は、[rust-sandbox](https://github.com/ShunichirouKamino/rust-sandbox)にて公開しています。

## ローカルに Rust の環境をセットアップする場合

- C++ Build Tools のインストール
  https://visualstudio.microsoft.com/ja/visual-cpp-build-tools/
- Rust のインストール
  https://www.rust-lang.org/tools/install
- VSCode に拡張プラグインのインストール
  - "matklad.rust-analyzer"
  - "vadimcn.vscode-lldb"
- ツールチェーンにて LSP 等のセットアップ
  - rustup component add rust-src
  - rustup component add rust-analysis
  - rustup component add rls

## Remote Container で実施する場合

- VSCode に拡張プラグインのインストール
  - ms-vscode-remote.remote-containers
- コマンドパレットから、以下を選択することで`.devcontainer/devcontainer.json`が生成される
  - `Remote-Containes: Add Development Container Configuration Files...`
  - `Rust`
  - `buster`
- VSCode 左下、`Open a Remote Window`から Remote Container をオープンする

# 写経に適したチュートリアルの紹介

-
