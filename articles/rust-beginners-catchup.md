---
title: "なぜRustなの？と言われた時のために"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rust"]
published: true
---

# 20 秒で概要

当記事では、Rust における以下の 4 つのいいところを特徴を紹介します。
他の言語と比較しながらコンセプトを学ぶことで、なぜ今 Rust を学ぶべきかを理解できます。

- Rust はメモリ安全な言語です
- Rust はリッチな型システムがあります
- Rust はエラー処理が分かりやすい
- Rust は健全なコミュニティの有るエコシステムです

また以下のような、Rust 学習における最初の一歩のお手伝いもします。

- 環境のセットアップ
- 写経に適したチュートリアルの紹介
- 躓きポイントの紹介

# Rust のいいところ

## Rust はメモリ安全な言語です。

### これまでのメモリ管理手法

プログラミング言語によるメモリ管理には、これまで 2 種類の方法が有りました。

- プログラマが全責任をもって管理する
  - 例）C 言語

```c
    char *str;
    int length = 100; // 100byte（半角文字100文字分）確保
    str = (char*)malloc(sizeof(char) * length);
    if (str == NULL)
    {
      .. // メモリの確保に失敗した場合の処理
    }
    free(str); // メモリの解放
```

- システムが GC によって自動で不要なメモリをかき集める
  - 例）Java, Python

### Rust におけるメモリ管理手法

Rust は、第三の方法でメモリ管理を行っております。
それが「所有権」という考え方で、以下のルールで成り立ちます。

- ① 値は、変数が束縛しており、変数のことを所有者と呼ぶ
- ② 値の所有者は、その瞬間は 1 つの変数のみ
- ③ 所有者である変数のスコープを抜けた際に、その値は利用不可能になる
- ④ 借用という考え方により、所有権を貸し出すことができます

Rust では、このルールがあることで、プログラマ自身がメモリの管理をすることなく、かつ GC が無いにも関わらずメモリを利用することができます。

#### 例１ 関数外スコープにおける変数へのアクセス

以下の例で、"hello"の所有者は`s`です。
`s`のスコープは`f()`内のため、ルール ③ により、`f()`の外では`s`にアクセスできません。

これは、スコープの概念を持つ言語、例えば Java でも同様の動きをします。

```rust
fn f() {
    let s = "hello".to_string();
    // sを使った処理
}
// ここではsにアクセスすることはできない
```

#### 例２ 関数に値を直接渡しした際の所有権のはく奪

以下の例で、`let s = "hello";`の時点で"hello"の所有者は`s`です。
`get_length`に`s`を渡すと、s の所有者は get_length 内の`s`に移ります。
ルール ② により、`get_length`以降`f()`では`s`にアクセスすることはできません。

```rust
fn get_length(s: String) -> usize {
    s.len()
}

fn f() {
    let s = "hello".to_string();
    let len = get_length(s);
    // ここではsにアクセスすることはできない
}
```

#### 例３ 別の引数に値を代入した際の所有権のはく奪

別の変数に代入した場合も、ルール ② により"hello"の所有者が移ります。これを、Rust では`move`と呼びます。

```rust
fn f() {
    let s = "hello".to_string();
    let s2 = s;
    // ここではsにアクセスすることはできない
}
```

#### 例４ 所有権の借用

ルール ④ の借用により、所有権を read-only でレンタルすることができます。
借用は、`s`が immutable の場合のみ（Rust ではデフォルトで変数は final）可能です。

```rust
fn get_length(s: String) -> usize {
    s.len()
}

fn f() {
    let s = "hello".to_string();
    let s2 = &s;
    // sにアクセスすることが可能
    let len = get_length(&s);
    // sにアクセスすることが可能
}
```

### 所有権の仕組みの何がすばらしいのか？

以下の例では、println の引数として`greet`を参照していますが、その前の`let greet2 = greet`の時点で所有権は`move`しています。
これにより、

- 実行する前にコンパイル時点でエラーを出力してくれるため、メモリ解放忘れによる実行中メモリリークのようなことは起きづらくなります。
- プログラマが手出しできない GC が無いため、全てのメモリリークの要因はコード上に存在します。

```rust
fn f() {
    let greet = vec!["Hello", "What's up?", "How is everything?"];
    let greet2 = greet;

    println!("{:?}", greet); // ここで静的コンパイルエラー
}
```

## Rust はリッチな型システムがあります

### struct

直積型です。C 言語でいう構造体と同じです。メソッドの無い class のイメージです。

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

fn f() {
    let taro = Person {
        name: String::from("taro"),
        age: 27,
    };
    println!("{:?}", taro); // Person { name: "taro", age: 27 }
}
```

### enum

列挙型です。Java の Enum と同じです。

```rust
#[derive(Debug)]
enum IpAddrKind {
    V4,
    V6,
}

fn f() {
    let v4 = IpAddrKind::V4;
    let v6 = IpAddrKind::V6;

    println!("{:?}, {:?}", v4, v6); // V4, V6
}
```

### 直和型

取りうるすべての型の羅列です。TypeScript では`a = number | string`のように表現され、Java では Java17 以降、`sealed`構文と`record`構文により実現されます。

```rust
/// Actionは、ToDoリストにおけるアクションを示す直和型です。
/// 複数のstructの列挙型で表現され、AddにもDoneにもListにもなれます。
pub enum Action {
    Add {
        text: String,
    },
    Done {
        position: usize,
    },
    List,
}
```

### trait

共通の振る舞いを定義します。struct に付与することで、クラスのような振る舞いが可能です。

```rust
struct Person {
    name: String,
    age: u8,
}

pub trait Judge {
    fn isOver30(&self) -> bool;
}

impl Judge for Person {
    fn isOver30(&self) -> bool {
        self.age > 30
    }
}
```

### 型システムが豊富なことは何が素晴らしいのか？

#### ドメイン知識を実装する幅が広がります

例えば Java では列挙型に対して、以下の制約が有ります。

- 変数が持てない
- 列挙値毎に定数の数は固定

```java
enum IpAddr {
    V4("127.0.0.1"),
    V6("::1"),

    private final String loopBack;

    public String getLoopBack(){
        return this.loopBack;
    }
}
```

Rust では、以下のように列挙値をそれぞれ別の型で表現できます。（直和型）
この仕様により、ドメイン知識を実装する幅が広がります。

```rust
enum IpAddr {
    V4 (u8, u8, u8, u8), // 8byte整数値を4つ持つタプル
    V6 { loopBack: String }, // Stringを1つもつstruct
}

fn f() {
    let v4LoopBack = IpAddr::V4(127, 0, 0, 1);
    let v6LoopBack = IpAddr::V6 {
        ip: "::1".to_string(),
    };
    println!("{:?}, {:?}", v4LoopBack, v6LoopBack); // V4(127, 0, 0, 1), V6 { ip: "::1" }
}
```

#### パターンマッチングによる分岐処理が簡潔に書けます

action の型（直和型のプリミティブな String, usize, 列挙定数）を判定し、後続の処理につなげます。

```rust
pub enum Action {
    Add {text: String, }, // 文字列変数を保持する構造体
    Done {position: usize, },  // usize型の数値変数を保持する構造体
    List,  // 列挙定数のみ
}

fn f() {
    let action = XXX::from_args(); // コマンドラインから何らかの値を取得

    match action {
        Add { text } => ..., // 文字列の場合の処理
        List => ... , // 何も指定されなかった場合の処理
        Done { position } => ... , // 数値の場合の処理
};
```

## Rust はエラー処理が分かりやすい

### Java におけるエラー処理

#### 検査例外

- 検査例外の Exception により、上位レイヤでキャッチします。
- 上位のモジュールでは、`try-catch`の記載が必須となります。

```java
public void any() {
    try {
        final var file = open("../input/input.json");
        // 何らかの処理
    } catch (IOException ex) {
        throw ex;
    }
}

public File open(String fileName) throws IOException {
    return new File(fileName);
}
```

#### 非検査例外

- 非検査例外の Exception により、Runtime 時の Exception を定義します。
  - `NullPointerException`, `ArrayIndexOutOfBoundsException`等
- 上位のモジュールでは、`try-catch`の記載が任意となります。

```java
public void any() {
    final var file = open("../input/input.json");
}

public InputStream open(String fileName) {
    try {
        return new FileInputStream(fileName);
    } catch (final FileNotFoundException e) {
        throw new UncheckedIOException(e);
    }
}
```

#### Java におけるエラー処理の課題

- 検査例外と非検査例外の使い分けとしては、検査例外は、「呼び出し側で発生を避けられないもの」です。しかし境界が曖昧で、例えば`NullPointerException`は非検査例外（呼び出し側で発生を避けられるもの）の定義ですが、実際は実行するまで見逃されることがほとんどです。
- 結果として、大原則である「Exception を握りつぶしてはいけない」について、実装時に見逃すことになります。上記の例で非検査例外である`NullPointerException`は、コンパイル時に検査できません。
- Exception の実装が、他の Class の実装と異なるためある程度学習コストがかかります。実際に Java を書いている人でも、Exception を何となく実装したり、Exception そのものの実装をしたことが無い人がほとんどではないでしょうか。

### VBA におけるエラー処理

VBA や C 言語のような例外機構が存在しない言語では、GoTo にてエラーハンドリングを行うことがあります。

```vb
Sub 実行()
    On Error GoTo Catch ' エラーが発生したら Catch の行へ処理を飛ばす
    Dim i As Integer
    i = "a"  ' エラー発生
    Exit Sub ' 正常に処理が行われたときに Catch: の処理を行わないように、ここで関数を抜ける
    Catch:
    ' エラー処理
End Sub
```

### VBA におけるエラー処理の課題

- GoTo 文は、何よりも先に解析されるため、非常に強力な構文です。結果として、処理がどこに飛ぶか分からないスパゲッティコードが生まれる原因となります。故に、GoTo 文を禁止する案件も少なくありません。
- そうはいっても、書き換えて関数呼び出しのネストが深い処理になってしまう場合や、メモリのクローズ忘れを防ぐ等、有用なシーンでの共通処理を定義しておくことができるため、完全には無くならないのが現状です。

### Rust におけるエラー処理

#### 検査例外

Rust の回復可能なエラー処理（検査例外）は、以下の 2 つの列挙型を返り値とすることで実現されます。

```rust
pub enum Result<T, E> {
    Ok(T),
    Err(E),
}

pub enum Option<T> {
    None,
    Some(T),
}
```

#### 非検査例外

Rust の回復不可能なエラー処理（非検査例外）は、`panic!`を発生させることで実現されます。panic!が発生すると、デフォルトではこれまで確保したメモリ等を自動的に開放していきます。

```rust
fn main() {
    panic!("crash and burn");  //クラッシュして炎上
}
```

#### 使い方 パターンマッチング

`Result`と`Option`が列挙型で実装されているということは、パターンマッチングの記法が使えます。

- `Result`は 2 つのジェネリクス定義が必要ですが、ほとんどの場合利用 Crate 内でラップされており、返り値側のみ指定すれば良いことがほとんどです。

```rust
pub fn add_task(task: String) -> Result<String> {
    // タスクの追加処理
}

pub fn any() -> Result<String> {
    let ret = match add_task("new task") {
        Ok(ret) => ret,
        Err(e) => return Err(e),
    };
    // retを使った処理
}
```

#### 使い方 ?によるシンタックスシュガー

`?`によるシンタックスシュガーが用意されており、`Result`を返す関数を複数呼び出しても、簡潔に記載することができます。

- `?`の役割は、「パターンマッチングを行ったうえで、`Ok` なら処理が進み、`Err` なら return する」という意味です。

```rust
fn add_task_from_file(file_name: String) -> Result<String, io::Error> {
    let mut f = File::open(file_name)?; // Resultが返却される関数
    let mut s = String::new();

    f.read_to_string(&mut s)?; // Resultが返却される関数
    add_task(s)?; // Resultが返却される関数

    Ok(s) // 最後に評価された式が返り値になり、returnの省略が可能
}
```

#### panic とパターンマッチングの併用

最終的に回復が不要と判断し、非検査例外とする場合は`panic!`を発生させるだけです。

```rust
pub fn add_task(task: String) -> Result<String> {
    // タスクの追加処理
}

pub fn any() -> Result<String> {
    let ret = match add_task("new task") {
        Ok(ret) => ret,
        Err(e) => {
            panic!("Tried to add task: {:?}", e)
        }
    };
    // retを使った処理
}
```

### Rust におけるエラー処理は何がうれしいのか？（まとめ）

- 検査例外と非検査例外の定義がはっきりしており、実装時に一目で分かる。
- エラーようの型が定まっていることで返り値にエラーを含むため、修復すべきエラーの握り潰しが発生しづらい。
- GoTo によるスパゲッティコードを産まず、保守性の高いコードが書ける。

## Rust は健全なコミュニティのあるエコシステムです

エコシステムは、一つの OSS を中心として、それを取り巻く生態系のことを指します。エコシステムの発達度と言語の発展は、少なからず依存関係が有ります。

- コンテナエコシステムで言うと、ランタイムは Docker や cri-o や Railcar 等が存在し、オーケストレーションでは k8s や Docker Compose、SaaS では ECS・EKS・GKE、今では Argo や Istio, velero 等の CRD もエコシステムに含まれます。
- Java のエコシステムでいうと、Maven や Gradle のビルドツールや、フレームワークである Spring boot, helidon, Quarkus、IDE や関連では Eclipse Foundation が有名で、AdoptOpenJDK が寄贈されたことも最近話題になりました。

### 用語の紹介

エコシステムの文脈で登場する用語について紹介します。

- LSP
  - language server protocol です。エディタや IDE が保有するソースを解析し、静的解析やフォーマット、自動補間などを行うバックエンドサービスとのプロトコルです。1 つの解析用 Language Server を実装することで、複数の IDE やエディタから利用されることが可能になりました。Microsoft さんが 2016 年に公開し、VSCode の発展に大きく寄与しています。
- Crate
  - クレートと読みます。Rust における 1 つのプロジェクト単位を指し、ライブラリでも有ります。`bin`形式と`lib`形式が有り、自作したライブラリは[crates.io](https://crates.io/)にて公開することができます。

### Rust におけるエコシステム

- コンパイラ
  - `rustc`でコンパイルができます。Java における`javac`や C 言語における`gcc main.c -o main`のようなものです。
- ビルドツール・パッケージマネージャ
  - `Cargo`により、プロジェクトのビルドからパッケージマネージまで全て行ってくれます。また依存ライブラリのダウンロードや、テスト、ドキュメント生成等も可能な便利ツールです。
- ツールチェーン管理
  - `rustup`により、`rustc`や`cargo`等のツール群を一式インストールしてくれます。また、`rust-analyzer`や`rls`といった LSP、`rust-fmt`といったフォーマッタ、`clippy`といった Linter ツールも管理されます。

### LSP 動作例

- `rls`を導入すると、マウスオーバーで定義の参照が可能になります。

![image](/images/rust-beginner/rust-rls1.gif)

- `rls`を導入すると、参照先のインライン表示が可能になります。

![image](/images/rust-beginner/rust-rls2.gif)

- `rls`を導入すると、フォーマッタが利用できるようになります。

![image](/images/rust-beginner/rust-rls3.gif)

- `rls`を導入すると、エラー箇所の静的解析を行ってくれます。

![image](/images/rust-beginner/rust-rls4.gif)

### その他エコシステム

- [microsoft/vscode-dev-containers](https://github.com/microsoft/vscode-dev-containers/tree/main/containers/rust)によりリモートコンテナ用の Image が提供されており、VSCode での環境構築が簡単にできます。

- [crates.io](https://crates.io/)により、ライブラリの公開が可能です。[npm](https://www.npmjs.com/)のようなものです。

# 環境のセットアップ

- 当記事に記載の VSCode 環境は、[rust-sandbox](https://github.com/ShunichirouKamino/rust-sandbox)にて公開しています。
  - 環境は、Windows10 を想定しています。
  - VSCode は必須です。
  - Docker は任意です。

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

- [The Rust Programming Language](https://doc.rust-jp.rs/book-ja/title-page.html)
  Rust 公式日本語版
- [Rust の最初のステップ](https://docs.microsoft.com/ja-jp/learn/paths/rust-first-steps/)
  Microsoft 社提供のチュートリアル

# 躓きポイントの紹介

## **Hello, world で登場するエクスクラメーションマーク**

- メタプログラミングの文脈で登場する、マクロという構文です。
- 関数と使い方が似ていますが。一言でいうと、プログラムをプログラミングする為の構文です。
- 関数と違い、引数を可変数取ることが可能です。

```rust
fn main() {
    let x = "Hello, world";
    println!("Hello, world");
    println!("{}", x);
}
```

- マクロの使い方を学ぶだけなら簡単ですが、マクロを自作したり、マクロの中身を理解しようとすると飛躍的に難易度が上がります。

## **返り値があるのに return が無い関数**

- Rust では、最後の式で評価された値が戻り値になります。
- 『式』は値を返し、『文』は値を返しません。
- 式には`;`が不要で、よく見ると最後の節は式になっています。

```rust
fn parse() -> Result<i32, ParseIntError> {
    let number = match "10".parse::<i32>() {
        Ok(number) => number,
        Err(e) => return Err(e),
    };
    Ok(number)
}
```

- よく見るのは返り値 Result に対して、最後が Ok で終わる関数です。

## **if let**

- デストラクチャリング（分割代入）と if 演算が同時に行われ、その後の処理の有無が決定します。

```rust
enum Foo {
    GRID { x: f64, y: f64 }
    POINT { x: f64 }
}

fn main() {
    let g = Foo::GRID {x: 0.1, y: 0.2};
    if let Foo::GRID {x, y} = g { // デストラクチャリングにより、分割代入が行われる
        println!("{}, {}", x, y); // 分割代入が成功した場合のみ、0.1, 0.2が出力
    }
    if let FOO::POINT {x, y} = g {  // POINTにgは代入できない
        println!("{}, {}", x, y); // ここの分岐には到達しない
    }
}
```

## クロージャ

- **|x| x + 1**
  - クロージャです。Rust におけるクロージャとは、Java におけるラムダ式であり、匿名関数です。
  - `JavaScript`におけるクロージャとは、関数とその関数が参照可能な変数スコープのことですが、Rust では少し意味が違うようです。
    - とはいえ以下の例では、匿名関数 f が束縛する変数範囲は main のスコープまでなので、全く遠い概念ではありません。

```rust
fn main() {
    let a = 50;
    let f = |x| x + a;
    println!("{}", f(10)); // 60
}
```

# 終わりに

良い Rust ライフを！
