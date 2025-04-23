# Cheapo-Request サンプル

このサンプルは『Programming Rust』第2版の第20章「非同期プログラミング」で紹介されている、シンプルなHTTPクライアントの実装例です。`cheapo-request`は外部クレートに依存せずに、標準ネットワークAPIだけを使って簡易的なHTTPクライアントを実装しています。

## 概要

このサンプルでは、非同期プログラミングの基礎を理解するため、TCPソケットを使った簡単なHTTPクライアントを実装しています。URLを引数として受け取り、そのURLに対してHTTPリクエストを送信し、レスポンスを表示します。

## 学べる概念

- 非同期HTTPリクエストの基本原理
- 低レベルなTCPソケット通信
- Rustの非同期I/O処理
- HTTPプロトコルの基本構造
- エラーハンドリング

## 主要コンポーネント

### `cheapo_request` 関数

指定されたURLに対してHTTPリクエストを行う非同期関数です。

```rust
async fn cheapo_request(url: &str) -> std::io::Result<String> {
    // URLのパース
    let host = // URLからホスト名を抽出
    let path = // URLからパスを抽出
    let port = // ポート番号（通常は80）

    // TCPストリームの接続
    let mut stream = TcpStream::connect((host, port)).await?;

    // HTTPリクエストの作成と送信
    let request = format!("GET {} HTTP/1.1\r\nHost: {}\r\nConnection: close\r\n\r\n", path, host);
    stream.write_all(request.as_bytes()).await?;

    // レスポンスの読み取り
    let mut response = String::new();
    stream.read_to_string(&mut response).await?;
    
    Ok(response)
}
```

### メイン関数

コマンドライン引数からURLを受け取り、`cheapo_request`関数を呼び出して結果を表示します。

```rust
#[async_std::main]
async fn main() -> std::io::Result<()> {
    // コマンドライン引数の解析
    let url = std::env::args().nth(1)
        .expect("usage: cheapo-request URL");
    
    // HTTPリクエストの実行
    match cheapo_request(&url).await {
        Ok(response) => {
            println!("{}", response);
            Ok(())
        }
        Err(err) => {
            eprintln!("Error: {}", err);
            Err(err)
        }
    }
}
```

## 実行方法

```bash
# ビルド
cargo build

# 実行 (例: example.comにリクエスト)
cargo run -- http://example.com

# または直接バイナリを実行
./target/debug/cheapo-request http://example.com
```

## 学習ポイント

1. **基本的なHTTPクライアント**: 外部ライブラリに依存せず、TCPソケットを使って簡易的なHTTPリクエストを行う方法を学べます。

2. **URLのパース**: URLをホスト名、パス、ポート番号に分解する方法を理解できます。

3. **非同期I/O操作**: ネットワーク通信における非同期I/O操作の基本的な使い方を学べます。

4. **エラーハンドリング**: I/O操作中に発生する可能性のあるエラーの処理方法を理解できます。

5. **HTTPプロトコル**: HTTPリクエストとレスポンスの基本構造を理解できます。

## 制限事項

このサンプルは教育目的のシンプルな実装であり、以下の制限があります：

- エラー処理は基本的なものに限定されています
- HTTPSには対応していません
- リダイレクトには対応していません
- ヘッダーの詳細な解析は行っていません
- タイムアウト処理が実装されていません

## 発展課題

1. HTTPSサポートの追加
2. リダイレクト処理の実装
3. タイムアウト機能の追加
4. ヘッダーの詳細な解析機能の実装
5. POSTリクエストのサポート
6. cURLなどの本格的なHTTPクライアントと機能を比較する 
