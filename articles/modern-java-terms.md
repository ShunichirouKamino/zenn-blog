---
title: "現代のJavaのコード規約を考える"
emoji: "😽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["java"]
published: false
---

# Java の革命

## Java 8

Java8 では、関数型プログラミングに影響を受け、`ラムダ式`と呼ばれる関数の直接呼出しが可能になりました。
ラムダ式を実現するために、`Functional Interface`が導入されました。
ラムダ式を利用することで、`Stream API`が実現されました。

### ラムダ式

ラムダ式のメリットは、以下の通りです。

- Stream API の引数として
