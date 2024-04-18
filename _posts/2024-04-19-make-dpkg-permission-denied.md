---
title: Makefile から呼び出した `dpkg` が "Permission denied" で失敗する
category: diary
locale: ja_JP
---

## 結論

- GNU Make 4.3 には、コマンドと同名のディレクトリが `$PATH` 直下に存在する場合、コマンドが正しく実行できない問題が有る。
- /usr/libexec に `$PATH` を通すべきではない。

## 問題

Ubuntu 22.04.4 LTS で Makefile から `dpkg` を呼び出すと、"Permission denied" と Error 127 で失敗する。同一のコマンドを同じシェルで直接実行すると成功する。

```shell
> dpkg --print-architecture
arm64

> make dpkg
dpkg --print-architecture
make: dpkg: Permission denied
make: *** [Makefile:25: dpkg] Error 127

> cat Makefile
...
.PHONY: dpkg
dpkg:
        dpkg --print-architecture
...
```

## 調査

実行されている `dpkg` が異なる可能性を考えて、`which` で実行ファイルのパスを調べたが、Makefile の中でも外でも同一だった。

```shell
> which dpkg
/usr/bin/dpkg

> make dpkg
which dpkg
/usr/bin/dpkg
```

また、`sudo make dpkg` は動作することを確認した。

## 原因

GNU Make 4.3 には、コマンドと同名のディレクトリが `$PATH` 直下に存在する場合、コマンドが正しく実行できない問題がある。Stack Overflow に `docker` コマンドにおける同様の報告が有る。

[makefile - make: docker: Permission denied - Stack Overflow](https://stackoverflow.com/a/72646736/13474335)

> You probably have a directory name docker on your PATH. There's a bug in the current versions of GNU make (actually, it's a bug in gnulib) where it doesn't skip subdirectories of directories on the PATH.

私の環境では想定される実行ファイルの在処を片っ端から `$PATH` に入れていたため、/usr/libexec/dpkg が上記の条件に引っ掛かっていた。

## 対策

GNU Make の問題は GNU Make 4.4 で修正されているらしい。

ただ、そもそも /usr/libexec は直接実行されたくないものを置く場所らしく、`$PATH` を通すこと自体が想定されていないことが分かった。

[fhs - What is the purpose of /usr/libexec? - Unix &amp; Linux Stack Exchange](https://unix.stackexchange.com/a/386015)

> if something is put in /usr/libexec/ it's a clear indication that it's considered an internal implementation detail, and calling it directly as an end user isn't officially supported.

何故 /usr/libexec を `$PATH` に追加しようと思ったのかは最早記憶に無いが、それは即ち使っていないということ。/usr/libexec を `$PATH` から外すことで、無事 Makefile から `dpkg` コマンドを実行できるようになった。