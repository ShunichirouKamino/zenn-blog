---
title: "GitHubとZennの連携テスト"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ポエム", "GitHub", "Zenn"]
published: false
---

# GitHub と Zenn の連携テストです

## CLI のインストール

```console
$ npm init --yes # プロジェクトをデフォルト設定で初期化
$ npm install zenn-cli # zenn-cliを導入
```

## Zenn 用のセットアップを行う

Zenn を管理したいフォルダで以下を実施します。

```console
$ npx zenn init
```

- 記事作成のために、articles というフォルダが作られます
- 書籍作成のために、books というフォルダが作られます

## 記事の作成

```console
$ npx zenn new:article
# 記事のURLの一部となるslugを指定して作成することもできます。
$ npx zenn new:article --slug my-awesome-article
```
