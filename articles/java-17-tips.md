---
title: "Java17で導入すべき構文"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Java"]
published: false
---

# 30 秒で概要

## 主要な OpenJDK の Java11 の LTS 及び Java17 の提供開始

### [AdoptOpenJDK(Eclipse Adoptium)](https://adoptopenjdk.net/)

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

-

- **この記事で目指す姿**

![moverview](/images/gcp-gke-loggin/overview.png)
