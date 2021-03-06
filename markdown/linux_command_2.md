<!--
title:   【Linux】中級コマンド一覧
tags:    Linux,command,console
id:      1fde05b2c9fae91b4311
private: false
-->

# 初めに

前回は基本的なコマンドについて記事にしましたが今回は中級レベルのコマンドを記事にします。

# コマンド

## grep

ファイルまたは入力ストリームから、表現に一致する行を表示する。

/etc/password ファイルの中にある test というテキストを含む行を表示する。

```console
grep test /etc/password
```

## less

ファイルの内容が一度に一画面ずつ表示されます。

大量の出力するコマンドの出力を確認する場合にとても便利です。

```console
less /usr/share/dict/words
```

また、`grep`と`less`を組み合わせることをできます。

```console
grep ie /usr/share/dict/words | less
```

## pwd

カレント・ワーキング・ディレクトリの名前を出力をします。

```console
$ pwd
/Users/test
```

## diff

２つのテキストファイルの差分を知ることができます。

```console
diff file1 file2
```

## file

ファイル形式を表示します。

```console
file file
```

## find

`dir`内にある`file`を探すことができます。

```console
find dir -name file -print
```
