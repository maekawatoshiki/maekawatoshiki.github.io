---
title: "色々なツールを研究で使う際のtips"
date: 2025-11-22T15:33:00+09:00
draft: false
toc: true
images:
tags:
  - research
---


**これは，[名工大 Advent Calendar 2025](https://qiita.com/advent-calendar/2025/nitech) - 11日目の記事です．**

---

世の中には、ソフトウェア・エンジニアリングを行う上で役立つツール<small>（ツールと言わないものも含む）</small>が様々あります。

この文書では、それらのツールを研究で使う際のTipsを紹介します。

きっと、研究に関係なく役立つ内容も多いかと思います。

<!-- 特に、一般的な使い方とは異なる、研究活動ならではの使い方に焦点を当てます。 -->


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

## git の扱い方

### git の操作に自信がない場合は GUI で操作しよう

私は普段コマンドラインから git を操作していますが、わざわざそうする理由は特にありません。

エディタに Visual Studio Code などを使っている方は、付属の git 操作機能を積極的に使うのが良いでしょう。
きっと良いプラグインもあるでしょうし。

**これ以降は、CLI で git を操作する際の tips を紹介します。**

### 何かがおかしいと思ったら `git status --ignored`

ビルドが上手くいかないなど、リポジトリの状態に違和感を覚えた時は `git status --ignored` を確認してみましょう。

通常の `git status` は（`.gitignore` により）無視されているファイルを表示しませんが、
`--ignored` オプションを付けると無視されているファイルも表示されます。

気付かぬうちに生成物や実験結果などがコミット対象になっていることはよくあります。\
そういった際は、まず落ち着いて、リポジトリがクリーンな状態かを確認しましょう。

### 過去の変更を追うなら `git blame -C -M`

ある行の変更履歴を、移動やコピーも含めて追う場合は `git blame -C -M <file>` が便利です。

リファクタリングを繰り返したリポジトリであっても、ある行の起源を辿ることが容易になります。

それぞれのオプションの意味は以下の通りです。
- `-M`: 同じファイル内での行の移動・コピーを追跡する
- `-C`: 別ファイルからの移動・コピーを追跡する

### マージは non fast-forward で行う

ブランチのマージは non fast-forward で行うことをおすすめします。\
`git merge --no-ff` でマージするか、グローバル設定を変更（`git config --global merge.ff false`）しておきましょう。

Non fast-forward マージはマージコミットを作成するため、マージが行われたことを明示的に履歴へ残すことができます。\
また、これによりマージ単位のrevertが容易になります。


### 複数ブランチを同時に扱うなら `git worktree`

複数ブランチ間を頻繁に行き来したり、同時に複数のブランチを触りたい場合は `git worktree` を使いましょう。\
一つのリポジトリについて複数の作業ツリーを同時に扱うことができます。

似たことは複数回の `git clone` & `git switch` でも実現できますが、`git worktree` は以下の点で優れています。

- リポジトリの実体は一つだけ
    - 親となるディレクトリにしか `.git` が存在しないため、作業ツリーを増やしてもあまり容量を食いません
    - `git fetch/remote/gc` といった操作を一度行えば、すべての作業ツリーに反映されます
- 一つのブランチは一つの作業ツリーにしかチェックアウトできない
    - 特定ブランチが複数のディレクトリでチェックアウトされることはないため、誤った編集を防げます

### `git stash` を避ける

`git stash` は気軽に使える反面、「stash したまま忘れる」ことがあるので避けた方がよいです。
メッセージを付けたとしても、大抵役に立ちません（主観）。

別の作業をしたいときは `git worktree` を使うか、素直に（ブランチを切って）Work in progressとしてコミットしましょう。\
コミットしておけば、簡単に remote へ push することもできます<small>（後から force push で取り消す必要が生じますが）</small>。

### `git add -pu` でファイルをステージングする

誤った変更をコミットしないために、`git add -pu` を使って対話的に変更をステージングしましょう。

各オプションの意味は以下の通りです。

- `-p` (`--patch`): 変更を一ブロック（hunk）ずつ確認しながらステージングする
- `-u` (`--update`): 追跡されているファイルのみ対象にする

これらのオプションを使うことで、ファイルに複数種類の変更が混じっていても（リファクタとバグ修正など）、
意味のある粒度でコミットを分けることができます。

<!-- ### `--ours` / `--theirs` -->
