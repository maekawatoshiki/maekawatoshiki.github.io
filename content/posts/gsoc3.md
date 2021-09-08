---
title: "Google Summer of Code 2021 を終えました"
date: 2021-09-09T00:08:57+09:00
draft: false
toc: false
images:
tags:
  - gsoc
---

# TL;DR

- Google Summer of Code 2021 を終えました
  - First, Final Evaluation ともに通りました、という意味
- LLVM むずかしい
- [成果](https://docs.google.com/document/d/1ASGejJIL7rzEDwX2qZZqT4ATR2x3sd_aOTuMSCh2YTE/edit#heading=h.qv3zfxev18ag)

# Google Summer of Code とは

[前に書いた記事](../gsoc1)を参照してください。

# 何をしたのか

2021/May/20 から 2021/Aug/28 くらいまで、メンターの方たちと共に LLVM へ貢献していました。
（まだマージされていないパッチがあるんですけどね...）

正確には、コーディングをはじめたのは６月くらいからみたいです。

# LLVM?


> The LLVM Project is a collection of modular and reusable compiler and toolchain technologies. 
> [The LLVM Compiler Infrastructure](https://llvm.org/)

clang や lld などを傘下に持つ、コンパイラ基盤のプロジェクトです。

（以下 LLVM についてある程度詳しいことを前提として話を進めていきます；またいつか LLVM 自体の解説もしたい）

# テーマ

提出した Proposal のタイトルは **Utilize LoopNest Pass** でした。

Pass というと、Module Pass, Function Pass, Loop Pass, ... など色々あります。
その中に、最近追加されたばかりの **LoopNest Pass** (New Pass Manager でのみ動作) というものがあります。
これは、ネストしたループを扱うための Pass なのですが、
まだほとんど使われていません（でした）。
私がこの夏にやったことは、LoopNest Pass として実装されている Pass を増やすことです。

Loop Pass とは別に LoopNest Pass が作られた理由にはいくつかあります。
例えば、ネストしたループ向けの最適化手法には、Loop Pass (や Function Pass) として実装するとコード量が増えてしまうものや、そもそも
うまく実装できないものが存在します。そのような最適化手法は、LoopNest Pass として簡単に実装できる可能性が高いです。

## LoopNest Pass と Loop Pass の違い

Loop Pass Pipeline *X* に、Loop Pass *A, B* が順番に並んでいるとします。
ここで、*X* に以下のループを流してみます。（*X* に `outerL` に対応する `Loop` オブジェクトを渡す、の意）

```
outerL:
  innerL:
    body
```

すると、まず `innerL` が *A, B* によって最適化され、その後に `outerL` が *A, B* によって最適化されます。
要するに、最内ループから最外ループへ向かって最適化が行われるというわけです。

特に問題なさそうに見えますが、これはあくまで個別のループに個別の最適化を施す場合には問題がない、というだけです。

例えば、`outerL` と `innerL` を見比べてからループネスト全体に何らかの操作を施したい場合はどうしればいいのでしょうか。
（単純な解決法：`innerL` を無視して `outerL` の順番がやってきたらループネスト全体を見渡せば良い；
問題：`innerL` を飛ばすためのコードが必要になる。そもそも Loop Pass に渡される `Loop` オブジェクトはループネストを扱うのには適していない）

ここで LoopNest Pass の登場です。

Loop Pass Pipeline *Y* に、Loop Pass *A, B* と LoopNest Pass *C* が順番に並んでいるとします。
ここで、*Y* に先程のループを流してみます。

すると、まず `innerL` が *A, B* によって最適化され、その後に `outerL` が *A, B, **C*** によって最適化されます。
要するに、**LoopNest Pass は最外ループにのみ適用される**のです。
しかも、LoopNest Pass は `Loop` オブジェクトではなく `LoopNest` オブジェクトを受けとります。
これにより、Loop Pass として実装する場合と比べて、ループネストを扱いやすくなります。

## もうちょっと具体例: loop-interchange

以下のようなループネストを考えます。

```
for k = 0, 10:
  for i = 0, 10:
    for j = 0, 10:
      A[i, j, k] += B[i, j, k]
```

見るからに何かがおかしいですよね？最外ループの添字が `k` となっていますが、これではキャッシュに当たる気がしません。

このようなループネストには、それぞれのループの順番を入れ替える loop-interchange という最適化を施すと良いです。

```
# loop-interchange 後
for i = 0, 10:
  for j = 0, 10:
    for k = 0, 10:
      A[i, j, k] += B[i, j, k]
```

このような最適化は、ループネスト全体を見渡す必要があります。そのため LoopNest Pass として実装すると良いです。
（でないと、for-k を飛ばして・for-j を飛ばして、という処理を個々の Loop Pass 内に記述する必要が出てきます）

# 何を学んだのか

LLVM という大きな OSS プロジェクトに貢献する中で、色々なことを学びました。

例えば、私はいつも大きなソースツリーを見ると上から下まで読みたくなるのですが、
そんなことを LLVM に対してしていたら一生が終わってしまいます。
なので今回の GSoC ではそんなことはせず、自分の担当する部分に関連する部分だけを集中的に読みました。
大きなソフトウェアはその一部分のみを理解しているだけで十分（であることが望ましい…）なんだと実感しました。

もう一つ、痛感したことがあります。
それは、コーディング能力は確かに重要だけれども、それ以上にコミュニケーション能力が重要だということです。
パッチを投げる、レビューを受ける、変更の意図を説明する…、すべての過程で（英語での）わかりやすい説明を求められます。
自分の意図を正確に伝えられないと、相手に無駄な時間を取らせてしまうこともあります。

英語でのテキスト（&できれば口頭での）コミュニケーション能力は身につけておかないと痛い目に合います。

# どうでもいいこと一覧

- メンター２人（１人とはあんまり話さなかったけど）、プロジェクトメンバー２人、私の合計５人でプロジェクトを進めた
  - 私とプロジェクトメンバーの１人がほとんどのパッチを作ったと思う
- 連絡は Discord で行った
- 毎週 Webex でミーティングを行った。口頭での英語にほんの少しだけ慣れた。
- LLVM のコミット権をもらえて嬉しかった
- 2700USD 貰った （[国ごとに貰える量は違うみたい](https://developers.google.com/open-source/gsoc/help/student-stipends)）
- お金は [Payoneer](https://www.payoneer.com/ja/)ってやつで受け取った
- CI が落ちると怖い（メールが来る）
- Address/Memory Sanitizer は偉い
- 初めて master にコミットしたときは手が震えた
  - そしてそのコミットを何度も revert する羽目になって辛かった


以上。来年以降に参加する人の助けになればいいなあ。

（質問があれば twitter@uint256_t へ DM とか投げてください）
