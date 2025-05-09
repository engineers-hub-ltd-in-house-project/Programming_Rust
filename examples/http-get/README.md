# HTTP GETクライアントサンプル

このサンプルは『Programming Rust』第2版の第18章「入出力」で紹介されている、`reqwest`クレートを使用してHTTPリクエストを行うコマンドラインプログラムです。

## 概要

このプログラムは、コマンドライン引数として与えられたURLに対してHTTP GETリクエストを行い、レスポンスの内容を標準出力に表示します。HTTPクライアントの実装には`reqwest`クレートを使用しています。

## 学べる概念

- 外部クレート（`reqwest`）の使用
- HTTP通信の基本
- コマンドライン引数の処理
- エラーハンドリング
- I/O操作（標準出力への書き込み）
- ビルド依存関係の管理

## 主要コンポーネント

### `main`関数

プログラムのエントリーポイントとなる関数です。コマンドライン引数からURLを取得し、HTTPリクエストを実行して結果を表示します。

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    // コマンドライン引数の処理
    let url = std::env::args()
        .nth(1)
        .ok_or("no URL given")?;

    // HTTP GETリクエストの実行
    let body = reqwest::blocking::get(&url)?
        .text()?;

    // レスポンスの表示
    println!("{}", body);
    
    Ok(())
}
```

## 実行方法

```bash
# ビルド
cargo build

# 実行（例：RustのWebサイトにアクセス）
cargo run https://www.rust-lang.org

# または直接バイナリを実行
./target/debug/http-get https://example.com
```

## 学習ポイント

1. **外部クレートの使用**: `reqwest`クレートを使ってHTTP通信を行う方法を示しています。これは、Rustのエコシステムを活用する典型的な例です。

2. **ブロッキングとノンブロッキング**: `reqwest::blocking`モジュールを使用して、同期的なHTTPリクエストを行っています。これにより、簡潔でわかりやすいコードになっています。

3. **エラーハンドリング**: `Result`型と`?`演算子を使用して、エラーを適切に処理・伝播する方法を示しています。

4. **トレイトオブジェクト**: `Box<dyn std::error::Error>`を使用して、様々な型のエラーを統一的に扱う方法を示しています。

5. **CLIアプリケーション**: コマンドライン引数を処理して、シンプルなコマンドラインアプリケーションを構築する方法を示しています。

## 発展課題

1. コマンドラインオプションをサポートする（ヘッダーの追加、タイムアウトの設定など）
2. HTTPメソッドを拡張して、POST、PUT、DELETEなどもサポートする
3. レスポンスヘッダーやステータスコードも表示する機能を追加する
4. JSONやXMLなどのレスポンス形式に応じた整形表示を実装する
5. 複数のURLを並行して処理する機能を追加する
6. プログレスバーを表示して、ダウンロードの進行状況を可視化する
7. レスポンスをファイルに保存するオプションを追加する 
