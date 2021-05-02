---
title: "Play with LLVM-IR, without LLVM"
date: 2021-05-02T22:04:08+09:00
draft: false
toc: false
images:
tags:
  - untagged
---

# TL;DR

[LLVMの代替実装](https://github.com/maekawatoshiki/vicis)をRustで作っています。(ただの趣味)
これを使って LLVM-IR とじゃれ合ってみよう、という記事です。

# LLVM-IRに触れる。LLVM抜きで。

インターネットで放浪してたら、このような記事を見つけました。
[RustのLLVM IRでプログラム分析ことはじめ - Toshihiro YAMAGUCHI’s Diary](https://toyamaguchi.hatenablog.com/entry/2019/12/22/080000)

私はこれまでに、rustcの吐くLLVM-IRを直接見たことがほとんどありませんでした。丁度よい機会なので、LLVMの復習がてら記事と同じようなことをしようとしたのですが、ただ真似るだけでは面白くない。

*そうだ、LLVMをRustで実装して、同じようなことをしよう*

というわけで、趣味として [LLVMをRustで再実装し始めました。(Vicisと名付けた)](https://github.com/maekawatoshiki/vicis)
この記事では Vicis を使って、LLVM-IRを、LLVM抜きで触っていきます。

## とりあえずプロジェクトを作る

記事の中では、モジュールに含まれる関数を列挙していました。これと同じことをやってみます。

```
cargo new viewer --bin # viewerというプロジェクトにします
```

プロジェクトを作ったら、`Cargo.toml`にVicisを追加しておきましょう。

```
...
[dependencies]
vicis = { path = "/path/to/vicis" }
```

## IRを読み込む

Vicisは現在、LLVM Assemblyの一部にしか対応できていません。(bitcode対応など頑張ります...)

`src/main.rs`を以下のように変更します。

```rs
use std::{env, fs::read_to_string};
use vicis::ir::module;

fn main() {
    let args: Vec<String> = env::args().collect();
    let file = &args[1];
    let asm = read_to_string(file).expect("failed to read .ll file");

    let module = module::parse_assembly(asm.as_str()).expect("failed to parse module");
    println!("{:?}", module);
}
```

これだけで、コマンドライン引数として渡した `*.ll`ファイルの中身をパースして、内部表現としての`Module`に変換できます。
さっそく動かしてみましょう。

## Hello World

RustでHello Worldを書いて、そのIRを見てましょう。`hello.rs`というファイルを用意します。

```rs
// hello.rs
fn main() {
	println!("hello world")
}
```

そして、`rustc`でコンパイルします。IRを出力する点に注意。

```
rustc +stable hello.rs -o hello.ll --emit llvm-ir
```

できあがった `hello.ll` を引数として渡してみましょう。

```
cargo run hello.ll 
```

おそらくとっても長いIRが出力されたと思います。成功です。

## 解析

長いIRを目grepするのは大変なので、関数名と関数の引数の数を出力するパスを作ってみます。

`main.rs`を以下のように修正します。

```rs
use std::{any::Any, env, fs::read_to_string};
use vicis::{
    ir::{function, module},
    pass::{AnalysisPass, PassManager},
};

fn main() {
    let args: Vec<String> = env::args().collect();
    let file = &args[1];
    let asm = read_to_string(file).expect("failed to read .ll file");

    let module = module::parse_assembly(asm.as_str()).expect("failed to parse module");
    let mut pm = PassManager::new();
    pm.add_analysis(FunctionPrinterPass);

    for (_, func) in module.functions() {
        pm.run_analyses_on(func);
    }
}

pub struct FunctionPrinterPass;

impl AnalysisPass<function::Function> for FunctionPrinterPass {
    fn run_on(&self, func: &function::Function, _result: &mut Box<dyn Any>) {
        println!(
            "Visit: {} (takes {} args)",
            func.name(),
            func.params().len()
        );
    }
}
```

パスマネージャとかの実装がまだまだ適当です。それはさておいて、実行すると以下のように表示されるはずです。

```
Visit: _ZN3std10sys_common9backtrace28__rust_begin_short_backtrace17h2840e0051141d5fcE (takes 1 args)
Visit: _ZN3std2rt10lang_start17h2fcf2a95611b6d93E (takes 3 args)
Visit: "_ZN3std2rt10lang_start28_$u7b$$u7b$closure$u7d$$u7d$17h642830c734a0a093E" (takes 1 args)
Visit: _ZN3std3sys4unix7process14process_common8ExitCode6as_i3217he311503d36a1dd10E (takes 1 args)
Visit: _ZN4core3fmt9Arguments6new_v117h8a113b20b098f88cE (takes 5 args)
Visit: "_ZN4core3ops8function6FnOnce40call_once$u7b$$u7b$vtable.shim$u7d$$u7d$17h32e55b47309d557aE" (takes 1 args)
Visit: _ZN4core3ops8function6FnOnce9call_once17h2daf67ca1036ee98E (takes 1 args)
Visit: _ZN4core3ops8function6FnOnce9call_once17hcf10f9084cb14a26E (takes 1 args)
Visit: "_ZN4core3ptr85drop_in_place$LT$std..rt..lang_start$LT$$LP$$RP$$GT$..$u7b$$u7b$closure$u7d$$u7d$$GT$17h74f0928264863a31E" (takes 1 args)
Visit: _ZN4core4hint9black_box17h5fe2e550e252fbc3E (takes 0 args)
Visit: "_ZN54_$LT$$LP$$RP$$u20$as$u20$std..process..Termination$GT$6report17h8ac75e0172be52beE" (takes 0 args)
Visit: "_ZN68_$LT$std..process..ExitCode$u20$as$u20$std..process..Termination$GT$6report17h63a57c0453f08cd5E" (takes 1 args)
Visit: _ZN5hello4main17h391fdbb81e062d23E (takes 0 args)
Visit: rust_eh_personality (takes 5 args)
Visit: _ZN3std2rt19lang_start_internal17hd5b67df56ca01daeE (takes 4 args)
Visit: _ZN3std2io5stdio6_print17hfdac4ecf8a146755E (takes 1 args)
Visit: main (takes 2 args)
```

ちゃんと関数名と引数の数が表示されました。成功です。

# 終わりに

Vicisはまだまだ実装されていない機能が沢山あります。Issue&PR いつでも歓迎です。