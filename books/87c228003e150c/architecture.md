---
title: "Axum & SeaORM & MVC"
free: true
---

# Locoのアーキテクチャ

Locoは、[このページ](https://loco.rs/docs/getting-started/axum-users/)にこう書かれています

> Loco is based on Axum, it is "Axum with batteries included", and is very easy to move your Axum code to Loco.

つまり、[Axum](https://github.com/tokio-rs/axum)がLocoのベースになっているということです。
同ページでは、Axumで作成したデモアプリをLocoにリプレイスするハンズオンが提供されています。

そのため、Axumで作成したものをリプレイスしたい場合は、
Locoを使用することが非常に望ましいです。

また、データベース周りについては、[`SeaORM`](https://www.sea-ql.org/SeaORM/)を使用しています。これも「RustでDBを扱うならこれ！」みたいな定番クレートなので、信頼性は高いです。

そのため、Locoは信頼性の高い2つのエコシステムが組み合わさって構築されています。

しかしそれだけでは、フレームワークとしては完成しません。
それにさらに付加価値がついています。

その付加価値というのが

- Railsのようにあらゆる設定を特定の一つのファイルにまとめることができる
- RailsのようなMVCモデルを採用し、ロジックとデータモデル、表示部分の分離を実現している
- 設定がめんどくさくてわかりづらくなりがちな、アプリ背後で実行される処理の設定が非常に楽になっている

のような、ただクレートを組み合わせただけでは得られない上記の付加価値こそ、
Railsを模したLoco特有の恩恵であると言えます。