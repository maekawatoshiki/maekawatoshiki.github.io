---
title: "Google Summer of Code 2021 に参加します"
date: 2021-05-19T15:56:20+09:00
draft: false
toc: false
images:
tags:
  - gsoc
---



# TL;DR

- Google Summer of Code に提出したProposalが採択されました
- 晴れてGSoCに参加できることとなりました
- 参加団体はLLVMです
- 嬉しい

# Google Summer of Codeとは

> Google Summer of Code is a global program focused on bringing more  student developers into open source software development. Students work  with an open source organization on a 10 week programming project during their break from school.
>
> (https://summerofcode.withgoogle.com/)

Google Summer of Code は、OSSプロジェクトにメンター付きで参加することができ、さらに Google から報酬ももらえるイベントです。
その名の通り夏（休み）に行われるのですが、アメリカ規準であって、日本の大学などでは普通に授業が行われています。
本来であれば、学部2年生の私にとっては参加が少し厳しいはずでしたが、今年は新型コロナの影響でプロジェクトに取り組む時間の制約が緩くなった (40h/w→17h+/w) ので、思い切って参加してみました。(大学もずっとオンラインで、家にこもりっぱなしですし。)

# Proposal採択までの流れ

3月の初めから、以下の作業を始めました。

## 参加団体を選ぶ

[公式サイト](https://summerofcode.withgoogle.com/organizations/)を見れば、様々な団体がGSoCに参加しているとわかります。
私は、ずっと [LLVM](https://llvm.org/) に貢献したかったので、無論それを選びました。

## 参加プロジェクトを選ぶ

各団体のほぼすべてが、学生向けのプロジェクト一覧を公開しているはずです。LLVM なら[ここ](https://llvm.org/OpenProjects.html)。
興味のあるプロジェクトは、初めのうちは複数個選んでおいた方がいいと思います。後になって、選んだプロジェクトが想像以上に難しいものだと判明すると、Proposalを書くのが大変ですし、不必要に自分を追い込んでしまいます。

## メーリングリストと奮闘する

興味のあるプロジェクトを選んだら、各団体のメンターと連絡を取ります。ここから先は各団体の指示に従うべきです。
私の場合は、メーリングリストでメンターの方にプロジェクトの詳しい説明を聞いていました。[このあたり](https://lists.llvm.org/pipermail/llvm-dev/2021-March/149058.html)から辿れます。

(英語のメールを書くのがかなり久しぶりだったので少し緊張しました。)

## Proposalを書く

各団体に提出するためのProposalを書きます。ネット上には過去GSoCに参加した方のProposalの例が沢山公開されています。

[もちろん私も公開します。](https://docs.google.com/document/d/1HkpPIgvZ0h7OQSzNN-8OFS71MqwKCmH4QmhKFqOjaxw)

Proposalは、ある程度書いたらメンターの人に見てもらって、アドバイスを貰ったりしました。

## (↑と同時並行で) 小さめのパッチを送る

Proposalの内容と関係のある[小さめのパッチ](https://reviews.llvm.org/D99149)を送ったりもしました。
自分の能力を示すためにも、Issueを立てたり、PRを投げたりするのは重要だと思います。

# Your proposal has been accepted!

May/18/2021 (JST) の深夜に、以下のメールが届きました。

<blockquote class="twitter-tweet"><p lang="ja"  dir="ltr">[報告]<br>一層精進していきます。 <a  href="https://t.co/2yuJePuOYi">pic.twitter.com/2yuJePuOYi</a></p>— uint256_t (@uint256_t) <a  href="https://twitter.com/uint256_t/status/1394347867628703746?ref_src=twsrc%5Etfw">May 17, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js"  charset="utf-8"></script> 

受験が終わった時のような気分でした。
頑張ります。

# 次回予告

丁度今日の深夜にメンターの方と話すので、何か面白い話題が生まれたら書きたい。

