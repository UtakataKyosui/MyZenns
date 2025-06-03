---
title: "Locoとは"
free: true
---

# Locoとは何か
バックエンドAPI開発において、非常に人気の高い
`Ruby on Rails`のシステムを`Rust`で扱えるようにしたフレームワークです。

[公式ページ](https://loco.rs/)もめちゃくちゃ堂々と
**It’s Like Ruby on Rails, but for Rust.** と書いてます。

## `Ruby on Rails`との違い
- まずそもそも言語が違う
	- インタプリタを用いた動的型付け言語のRubyから、コンパイラを用いる静的型付け言語のRustへ
- パフォーマンスが違う
	- Loco公式の比較結果によるとLocoの方がRuby on Railsと比べて、10倍以上のパフォーマンスを記録している

## セットアップ
Rustが入っていることを前提とすると、まずこれらを導入する必要があります。
```sh
cargo install loco # LocoのCLI自体はこれ
cargo install sea-orm-cli # DBを扱うならこれが必要
```

これらを導入後、以下のコマンドを実行します。
```sh
loco new
```