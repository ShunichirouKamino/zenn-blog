---
title: "Java17で導入すべき構文"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Java"]
published: false
---

# 30 秒で概要

## 主要な OpenJDK の Java11 の LTS 及び Java17 の提供開始

### [Eclipse Adoptium(旧 AdoptOpenJDK)](https://adoptopenjdk.net/)

- Java11 の LTS：2024 年 10 月（[AdoptOpenJDK Support](https://adoptopenjdk.net/support.html)）
  - ※できる限りその後もサポートするよと言っているが、基本はここを EOS と考えるのが自然
- Java17 からは[Eclipse Adoptium](https://adoptium.net/)から提供される。
  - [背景 - AdoptOpenJDK to join the Eclipse Foundation](https://blog.adoptopenjdk.net/2020/06/adoptopenjdk-to-join-the-eclipse-foundation/)
    - London Java Community CIC では手に負えなくなってきており、Java エコシステムの中核とも言える EclipseFoundation に合流することで、更なる発展を目指したいとの意向。
- Java17 の LTS：未記載。ただしリリースから最低 4 年間を謳っており、リリースが 2021 年 10 月 19 日なので、2025 年同日までは少なくともサポートされる。（[Adoptium Support(https://adoptium.net/support.html)]）

### [Microsoft Build of OpenJDK](https://docs.microsoft.com/ja-jp/java/openjdk/download)

- [Microsoft Build of OpenJDK のサポート ロードマップ](https://docs.microsoft.com/ja-jp/java/openjdk/support)
  - Java11 の LTS : 少なくとも 2024 年 9 月
  - Java17 の LTS : 少なくとも 2027 年 9 月

### [Amazon Corretto](https://aws.amazon.com/jp/corretto/)

- [Amazon Corretto のよくある質問 - Corretto のサポートカレンダー](https://aws.amazon.com/jp/corretto/faqs/#support)

  - Java11 の LTS : 2027 年 9 月
  - Java17 の LTS : 2029 年 10 月

## 追加機能まとめ

- [OpenJDK - JDK 12](https://openjdk.java.net/projects/jdk/12/)
- [OpenJDK - JDK 13](https://openjdk.java.net/projects/jdk/13/)
- [OpenJDK - JDK 14](https://openjdk.java.net/projects/jdk/14/)
- [OpenJDK - JDK 15](https://openjdk.java.net/projects/jdk/15/)
- [OpenJDK - JDK 16](https://openjdk.java.net/projects/jdk/16/)
- [OpenJDK - JDK 17](https://openjdk.java.net/projects/jdk/17/)

# 導入したい構文

JDK17 の時点で、正式リリースされた構文のみ紹介してます。

## Java16

### [JEP 394: Pattern Matching for instanceof](https://openjdk.java.net/jeps/394)

明示的なキャスト時に、これまでは慣用として以下の構文を利用していました。これは、`obj` が `String` でない場合の`ClassCastException`を防ぐためです。

```java
if (obj instanceof String) {
    String s = (String) obj;
    ...
}
```

`instanceof` 時に同時にオブジェクトの形状比較（パターンマッチング）が行われ、キャスト後の変数`s`がスコープ内で利用できるようになりました。

```java
if (obj instanceof String s) {
    // Let pattern matching do the work!
    ...
}
```

### [JEP 395: Records](https://openjdk.java.net/jeps/395)

`record`クラスは、以下の目標として実装されました。

- 冗長な Java において、以下のモチベーションでコードの簡略化を行う
  - 不変なデータのモデリングを行う
  - equals 等のメソッドを自動で実装する

これまでの`Bean`の実装には、変数を 2 つ持つだけでも以下のような冗長なコードの記載が必要でした。

```java
class Point {
    private final int x;
    private final int y;

    Point(final int x, final int y) {
        this.x = x;
        this.y = y;
    }

    int x() { return x; }
    int y() { return y; }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        Point other = (Point) o;
        return other.x == x && other.y == y;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }

    @Override
    public String toString() {
        return String.format("Point[x=%d, y=%d]", x, y);
    }
}
```

record クラスを用いると、上記の`Bean`が以下の実装となります。

- `record`クラスは、明示的に`final`となります。
- 各要素の private final が自動実装されます。
- getter として、`int x()`, `int y()`が自動実装されます。
- `Canonical Constructor`が自動実装されます。
  - その他、`Compact Constructor`を定義可能です。（後述）
- `equals()`及び`hashCode()`が自動実装されます。
- `toString()`が自動実装されます。

```java
record Point(int x, int y) {}
```

#### コンストラクタ

- `Canonical Constructor`
  自動実装されるコンストラクタです。変数への代入のみの役割を持ちます。
  上記 Point record であれば、以下のコンストラクタが自動生成されます。

```java
    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
```

- `Compact Constructor`
  コンパクトコンストラクタには、検証等を行うコードのみを記載します。
  その他のフィールドへの値を代入する初期化コードは、実装する必要はありません。
  例えば、それぞれが正の値である必要が有る場合は、以下のようにバリデーションを実装します。
  引数は、`record`クラスを明示する際に指定しているため、コンストラクタ引数は不要です。

```java
    record Point(int x, int y) {
        Point {
            if (x < 0) {
                throw new ...
            }
            if (y < 0) {
                throw new ...
            }
        }
    }
```

- `Factory`
  ファクトリパターンの実装も可能です。
  ファクトリパターンは、Java には存在しないデフォルト引数を実現するために有意義です。
  以下は、y を固定した（デフォルトとした）ファクトリと、x,y を受け取るファクトリを実装したレコードクラスの例です。

```java
    record Point(int x, int y) {
        public static Point of(final int x) {
            final var y = 1;
            return new Point(x, y);
        }

        public static Point of(final int x, final int y) {
            return new Point(x, y);
        }
    }
```

## [JEP 409: Sealed Classes](https://openjdk.java.net/jeps/409)

`Sealed Classes`は、DDD の文脈におけるドメイン知識をコードに落とし込む際に重要な役割を果たします。
Java には元々、現実世界のドメインを明示的に表現するために便利な構文として、`enum`が存在します。

```java
enum Shape { CIRCLE, RECTANGLE, SQUARE }

Shape shape = Shape.CIRCLE;
switch (shape) {
    case CIRCLE: ...
    case RECTANGLE: ...
    case SQUARE: ...
}
```

これは、`Shape` を単純なコード値として分岐処理を設ける際には、可読性の側面で大きな威力を発揮します。
しかし、CIRCLE, RECTANGLE, SQUARE には別々の処理（メソッド）を持たせたい場合が多いです。
そういった場合は、interface を用いて以下のような実装となります。

```java
    interface Shape {...}
    final class Circle implements Shape {...}
    final class Rectangle implements Shape {...}
    final class Square implements Shape {...}
```

`Shape`を実装することで、`Shape`の持つドメイン知識を継承し、それぞれのクラスにふるまいを持たせることが可能です。
しかし致命的な問題が一点あります。この実装では、このアプリケーション上`Shape`には`CIRCLE`, `RECTANGLE`, `SQUARE`の 3 種類しか存在しないということを表せていません。

ここで登場するのが、`Sealed Classes`です。
Class とありますが、interface でも実装可能です。

- `Sealed`は、`Shield`ではなく、封じられたという意味合い。
- `permits`句にて指定したクラスで実装されていない場合は、`sealed`クラス側にコンパイルエラーが発生する。
- `permits`句にて指定したクラス以外で実装された場合は、実装クラス側にコンパイルエラーが発生する。

```java
    public sealed interface Shape permits Circle,Rectangle,Square {...}
    final class Circle implements Shape {...}
    final class Rectangle implements Shape {...}
    final class Square implements Shape {...}
    }
```

これは、特に`record`との組み合わせでうまく機能します。
`sealed`と`record`の組み合わせは、代数的データ型と呼ばれます。

# 付録

## 代数的データ型

以下 3 つのデータ型を合わせて代数的データ型と言います。

- 列挙型
- 直積型
- 直和型

### 列挙型

列挙型は、同一の型の羅列です。列挙できることは、言い換えると`switch`可能ということを表しています。
Java では、`enum`で表現されます。同一の型の値を列挙し、種類を区別します。

```java
    enum Color {
        BLUE, RED, GREEN,
    }
```

### 直積型

直積型は、型の制限です。これまでの Java で表現することは難しく、C 言語では、構造体がこれに該当します。
Employee の取りうる値の範囲は、$(intの範囲 * char[51]の範囲 * intの範囲)$で表現できるからと理解しています。

```c
struct Employee {
        int number;     /* 従業員番号 */
        char name[51];  /* 氏名 */
        int salary;     /* 給与 */
};
```

Java で`record`が導入されたことにより、これを表現することが可能になりました。

```java
    record Employee(int number, String name, int salary) {}
```

### 直和型

列挙型と直積型両方の特徴を持っています。つまり、

- switch 可能
- 要素を複数持つことが可能

となります。これは、Java で`slealed`が実装されたことで実現されました。

```java
    sealed interface Person permits Teacher,Student {}
    record Teacher(int serviceYears, String name, int salaly) implements Person {};
    record Student(int age, String name) implements Person {};

    final var student = new Student(15, "Taro");
    final var teacher = new Teacher(3, "Hanako", 5_000_000);
```
