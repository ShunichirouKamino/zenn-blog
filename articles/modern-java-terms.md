---
title: "現代のJavaのコード規約を考える"
emoji: "😽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["java"]
published: false
---

# ラムダ式

## Java 8

Java8 では、関数型プログラミングに影響を受け、`ラムダ式`と呼ばれる関数の直接呼出しが可能になりました。

- ラムダ式を実現するために、`Functional Interface`が導入されました。
- ラムダ式を利用することで、`Stream API`が実現されました。

## ラムダ式

ラムダ式のメリットは、以下の通りです。

- 一度しか利用しないクラス、いわゆる匿名クラスを記載する必要なく、一時的にクラス内の関数を利用可能とする。

**一時的に関数を利用する場合の、Java7 以前の記載方法**

- test 関数のスコープで一時的に Greet をインスタンス化し、関数も Override することで、インスタンシエート以降で利用可能となる。

```java
    @Test
    void test() {
        final Greet greetHello = new Greet() {
            @Override
            public void print(final String greetString) {
                System.out.println(greetString);
            }
        };
        greetHello.print("hello!");
    }

    public interface Greet {
        void print(String greetString);
    }
```

**一時的に関数を利用する場合の、Java8 以降の記載方法**

- 一度しか利用しないクラスのインターフェースが 1 つのメソッドのみを持つ場合のみ、インスタンス化を行わずに関数定義を行うことができる。
- この記法のことを`ラムダ式`と呼ぶ。

```java
    @Test
    void test() {
        final Greet greetHello = greetString -> System.out.println(greetString);
        greetHello.print("hello!");
    }

    @FunctionalInterface
    public interface Greet {
        void print(String greetString);
    }
```
