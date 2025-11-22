---
title: "色々な技術を研究で使う際のtips"
date: 2025-11-22T15:33:00+09:00
draft: true
toc: true
images:
tags:
  - research
---

世の中には、ソフトウェア・エンジニアリングを行う上で役立つツール<small>（ツールと言わないものも含む）</small>が様々あります。

この文書では、各ツールを研究で使う際のTipsを紹介します。

特に、一般的な使い方とは異なる、研究活動ならではの使い方に焦点を当てます。


## シェルスクリプトの書き方

要約: スタイルガイドに従うこと、及び shellcheck を常用することを推奨します。

### スタイルガイドに従う

誤読のなく、ポータブルなシェルスクリプトを書くためには、
[Googler によるシェルスクリプトのスタイルガイド](https://google.github.io/styleguide/shellguide.html)は
必読だと思います。

以下に、私が重要だと思った部分をスタイルガイドから抜粋します。
<small>スタイルガイドにない項目もあるかも</small>。


### シェルとして `bash` を使う

Shebang は `#!/bin/bash` で始めましょう。

スタイルガイドはこう述べています。

> ... In particular, this means there is generally no need to strive for POSIX-compatibility or otherwise avoid “bashisms”.

POSIX 準拠を目指すとかなり書きづらくなるのは想像がつきます。
Bash は実質すべてのマシンで使えますから、このあたりが妥協点なのでしょう。


### `set -euxo pipefail` オプションを使う

Shebang の次の行に `set -euxo pipefail` と書きましょう。

`set` はシェルのビルトインコマンドであり、シェルのオプションを変更することができます。
オプションとは `-euxo pipefail` （`-e -u -x -o pipefail` と同じ意味）の部分を指しますが、それぞれの意味は以下の通りです。
以下の説明は荒いので、詳細は [4.3.1 The Set Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html) を参照してください。

- `-e`: コマンド実行が失敗した場合にスクリプトを直ちに終了する
- `-u`: 未定義の変数を参照した場合にエラーにする
- `-x`: 実行する寸前のコマンドを標準エラーに出力する
- `-o pipefail`: パイプで繋いだコマンド群のうち、どれか一つでも失敗した場合に全体を失敗とする

> 注意：Shebang を `#!/bin/bash -euxo pipefail` とするのは、意図した動作とならないため避けましょう。
> 
> そのように記述すると、例えば
> [Linux カーネルでは `/bin/bash` に一つの引数（`-euxo`）しか渡されず](https://github.com/torvalds/linux/blob/b236920731dd90c3fba8c227aa0c4dee5351a639/fs/binfmt_script.c#L81-L85)、
> `pipefail` は無視されてしまいます。


### 変数は `${var}` 記法で展開する

変数展開は `$var` ではなく `${var}` としましょう。\
ブレース `{` `}` で境界を明示すると読みやすく、変数名の終わりが明確になります。

（単文字の特殊変数 *`$?` や `$#` など* は例外です）

#### 変数名と後続の文字列を明確に区別できる

```bash
# ブレースなしだとエラー
prefix="test"
echo "$prefix_file"  # $prefix_file という変数を参照しようとしてエラー

# ブレースありで正しく動作
echo "${prefix}_file"  # test_file
```

#### デフォルト値の設定や未定義変数の検出にも使える

このほかにも、ブレース記法は文字列操作など色々な用途で使えます。

```bash
# 変数が未定義の場合にデフォルト値を設定
echo "${var:-default_value}"  # var が未定義なら default_value を出力

# 変数が未定義の場合にエラーにする
echo "${var:?var is not defined}"  # var が未定義なら `var is not defined` と出力
```


### `shellcheck` を使う

シェルスクリプトを書いたら、とりあえず `shellcheck` コマンドに通しましょう。

`shellcheck` はシェルスクリプトの静的解析ツールであり、上記のスタイルガイドに反する箇所や潜在的なバグを指摘してくれます。\
スクリプト実行前の安全装置として有用です。

最近のエディタは LSP 経由で `shellcheck` と連携できることが多いので、設定しておきましょう。

```bash
$ cat a.sh
#/bin/bash

echo "$var"

$ shellcheck a.sh

In a.sh line 1:
#/bin/bash
 ^-- SC1113 (error): Use #!, not just #, for the shebang.


In a.sh line 3:
echo "$var"
      ^--^ SC2154 (warning): var is referenced but not assigned.

For more information:
  https://www.shellcheck.net/wiki/SC1113 -- Use #!, not just #, for the sheba...
  https://www.shellcheck.net/wiki/SC2154 -- var is referenced but not assigned.
```

### 最後に：長い処理を書くなら他の言語を検討する

シェルスクリプトは便利ですが、複雑で長い処理を書くには向いていません。

以下のどれかに当てはまる場合は Python / Perl / Ruby / ECMAScript などの言語を使いましょう。

- 100行を超える
- 制御フローが単純でない
- 単なるラッパー以上のことをする
- パフォーマンスが重要
- 今後も保守する可能性がある
