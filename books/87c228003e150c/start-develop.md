---
title: "開発を始める -start_develop-"
---

# 開発をしてみる

## 開発対象

![Axumのサンプルコードリポジトリ](https://github.com/tokio-rs/axum/tree/main/examples)

今回は、Axum公式が公開している、TodoアプリのAPIのAxum実装のサンプルを
Locoでリファクタリングした上で、Locoに搭載されている高度な機能を使って、
より高機能なAPIとして開発していきたいと思います。

## コードリーディング

このサンプルのエンドポイント構成はこうなっています。

| パス | HTTPメソッド | 役割 |
| --- | --- | --- |
| `/todos` | GET | JSON形式で登録されているTodoを返す |
| `/todos` | POST | Todoの新規作成 | 
| `/todos/{id}` | PATCH | Todoの内容の更新 |
| `/todos/{id}` | POST | Todoの削除 | 

Todoデータを管理するDBに関しては、以下のように宣言されています。
```rust
type Db = Arc<RwLock<HashMap<Uuid, Todo>>>;
```

Todoデータの型はこうなっていました
```rust
#[derive(Debug, Serialize, Clone)]
struct Todo {
    id: Uuid,
    text: String,
    completed: bool,
}
```

いくつか見慣れなさそうなものがあると思うので、
軽くここの説明をしておくと

| 型 | 役割（ざっくり） |
| --- | --- | 
| `Arc` | 複数のスレッド間で所有権を安全に共有するためのポインタ |
| `RwLock` | Read-Writeのアクセス制御を行うもの | 
| `HashMap` | KV形式のデータ構造（そういう意味ではJSONに酷似） | 

という認識があればOKです。

このサンプルのプログラムは驚くほどシンプルで、
`main.rs`の中に書き切られている上で161行しかないのですが、
空行を切り詰めると150行を切るくらいコードが少ないです。

## リファクタリング上の仕様変更
Locoで実装する上で、以下の変更を行います。

- DBはメモリ上での擬似再現ではなく、PostgreSQLを利用する
- 機能として、Todoの通知機能を作成したいので期限を設定する
	- そのため、Todoのカラムに期限を設定するカラムが増える
	- 期限を超過していなくて、かつ未完了なものがあった場合、通知を飛ばすように作成
	- Locoには`Mailer`機能がありますが、今回は`Webhook`を使って、Discordに通知を飛ばす