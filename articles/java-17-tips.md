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

## java16

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

    Point(int x, int y) {
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

- 各要素の private final が自動実装されます。
- getter として、`int x()`, `int y()`が自動実装されます。
- `Default Constructor`が自動実装されます。
  - その他、`Compact Constructor`を定義可能です。（後述）
- `equals()`及び`hashCode()`が自動実装されます。
- `toString()`が自動実装されます。

```java
record Point(int x, int y) {}
```

#### コンストラクタ

- `Default Constructor`
  自動実装されるコンストラクタです。変数への代入のみの役割を持ちます。
  上記 Point レコードであれば、以下のコンストラクタが自動生成されます。

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
  ファクトリは、Java には

## [JEP 409: Sealed Classes](https://openjdk.java.net/jeps/409)

java は、現実世界のドメインを明示的に表現するために便利な構文として、`enum`が存在します。

```java
enum Shape { MERCURY, VENUS, EARTH }

Shape shape = ...
switch (shape) {
    case CIRCLE: ...
    case RECTANGLE: ...
    case SQUARE: ...
}
```

これは、`Planet` を単純なコード値として分岐処理を設ける際には、可読性の側面で大きな威力を発揮します。
しかし、本来は MERCURY, VENUS, EARTH には別々の処理（メソッド）を持たせたい場合が多いです。
そういった場合は、以下のような実装となります。

```java
interface Celestial { ... }
final class Planet implements Celestial { ... }
final class Star   implements Celestial { ... }
final class Comet  implements Celestial { ... }
```

`Celestial`（天体）を実装することで、
