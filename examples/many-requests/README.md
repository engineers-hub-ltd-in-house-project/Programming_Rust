# 非同期HTTP要求サンプル

このサンプルは『Programming Rust』第2版の第20章「非同期プログラミング」で紹介されている、単一スレッドで複数のHTTPリクエストを並行処理する例です。

## 概要

このプログラムは、`async-std`クレートを使用して、指定されたURLのリストに対して並行してHTTPリクエストを行います。シングルスレッドでの非同期処理のデモンストレーションとして、各リクエストの応答時間を記録して表示します。

## 学べる概念

- Rustの非同期プログラミング（`async`/`await`）
- `async-std`クレートの使用方法
- 単一スレッドでの並行処理
- HTTPリクエストの非同期処理
- `Future`の基本概念
- タスクの同時実行とオーケストレーション

## 主要コンポーネント

### `fetch_url`関数

指定されたURLに対してHTTPリクエストを行い、レスポンスを文字列として返す非同期関数です。

```rust
async fn fetch_url(url: String) -> Result<String, surf::Error> {
    let mut response = surf::get(url).await?;
    let body = response.body_string().await?;
    Ok(body)
}
```

### `cheapo_request`関数

下位レベルのネットワーク操作を使用して、HTTP GETリクエストを実装する関数です。`async-std`の`TcpStream`を直接使用しています。

```rust
async fn cheapo_request(host: &str, port: u16, path: &str) -> Result<String, Error> {
    // TCPコネクションの確立とHTTPリクエストの送信
    // ...
}
```

### メイン関数

複数のURLに対して並行してリクエストを行い、各リクエストの完了を待ち合わせます。

```rust
#[async_std::main]
async fn main() -> Result<(), Error> {
    // URLリストの作成
    let requests = vec![
        "https://example.com".to_string(),
        "https://example.org".to_string(),
        // ...
    ];
    
    // 各URLに対してタスクを生成
    let mut tasks = vec![];
    for url in requests {
        tasks.push(async_std::task::spawn_local(async move {
            match fetch_url(url.clone()).await {
                Ok(body) => println!("Received {} bytes from {}", body.len(), url),
                Err(err) => eprintln!("Error fetching {}: {}", url, err)
            }
        }));
    }
    
    // すべてのタスクの完了を待機
    for task in tasks {
        task.await;
    }
    
    Ok(())
}
```

## 実行方法

```bash
# ビルド
cargo build

# 実行
cargo run
```

実行すると、指定されたURLに対して並行してHTTPリクエストが行われ、各リクエストの結果（成功した場合はレスポンスサイズ、失敗した場合はエラー）が表示されます。

## 学習ポイント

1. **非同期プログラミング**: `async`/`await`構文を使用した非同期コードの書き方を学べます。

2. **単一スレッドでの並行処理**: `spawn_local`を使用して、単一スレッド上で複数のタスクを同時に実行する方法を示しています。

3. **外部クレートの活用**: `async-std`や`surf`などの外部クレートを使用して、非同期I/Oやネットワーク処理を行う方法を学べます。

4. **エラーハンドリング**: 非同期コンテキストでのエラー処理の方法を示しています。

5. **タスクの同期**: 複数のタスクを生成し、それらの完了を待ち合わせる方法を学べます。

## 発展課題

1. リクエストにタイムアウトを追加する
2. 同時実行数を制限するスロットリング機能を実装する
3. レスポンスのキャッシュ機能を追加する
4. リクエストに優先順位を付ける機能を実装する
5. 失敗したリクエストの自動再試行機能を追加する
6. ステータスコードに基づいた条件付き処理を実装する
7. マルチスレッド環境にも対応するために`spawn`を使用するバージョンを作成する 
