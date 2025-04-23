# ファイルコピーサンプル

このサンプルは『Programming Rust』第2版の第18章「入出力」で紹介されている、ファイルコピープログラムの実装例です。

## 概要

このプログラムは、コマンドライン引数として与えられた2つのパス（コピー元とコピー先）を使用して、あるファイルから別のファイルにデータをコピーする機能を提供します。標準ライブラリの`std::fs`と`std::io`モジュールを使用したシンプルなファイル操作の例です。

## 学べる概念

- ファイルの読み書き操作
- コマンドライン引数の処理
- エラーハンドリング
- バッファを使った効率的なI/O
- `std::fs`と`std::io`モジュールの使用方法

## 主要コンポーネント

### `main`関数

プログラムのエントリーポイントとなる関数です。コマンドライン引数を解析し、ファイルコピーを実行します。

```rust
fn main() -> std::io::Result<()> {
    // コマンドライン引数の解析
    let args: Vec<String> = std::env::args().collect();
    if args.len() != 3 {
        eprintln!("使用方法: copy 元ファイル 先ファイル");
        std::process::exit(1);
    }

    // ファイルコピー処理の実行
    copy_file(&args[1], &args[2])?;
    Ok(())
}
```

### `copy_file`関数

ファイルをコピーする処理を担当する関数です。

```rust
fn copy_file(source: &str, destination: &str) -> std::io::Result<()> {
    // 入力ファイルを開く
    let mut input_file = File::open(source)?;
    
    // 出力ファイルを作成
    let mut output_file = File::create(destination)?;
    
    // バッファを使ってデータをコピー
    let mut buffer = [0; 4096]; // 4KBのバッファ
    loop {
        let bytes_read = input_file.read(&mut buffer)?;
        if bytes_read == 0 {
            break; // ファイルの終端に達した
        }
        output_file.write_all(&buffer[..bytes_read])?;
    }
    
    Ok(())
}
```

## 実行方法

```bash
# ビルド
cargo build

# 実行（例：file1.txtの内容をfile2.txtにコピー）
cargo run file1.txt file2.txt

# または直接バイナリを実行
./target/debug/copy original.txt copied.txt
```

## 学習ポイント

1. **ファイルI/O操作**: `File::open`と`File::create`を使ってファイルを開く/作成する方法と、`read`と`write_all`メソッドを使ってデータを読み書きする方法を学べます。

2. **バッファリング**: 固定サイズのバッファを使って効率的にファイルデータをコピーする方法を示しています。

3. **エラーハンドリング**: `std::io::Result`と`?`演算子を使って、I/O操作中に発生する可能性のあるエラーを適切に処理する方法を学べます。

4. **コマンドライン引数**: `std::env::args()`を使ってコマンドライン引数を処理する方法を示しています。

5. **プログラム終了**: エラー状態を示すために`std::process::exit()`を使用する方法を学べます。

## 発展課題

1. 進捗表示（プログレスバー）を追加する
2. コピー速度の計測と表示機能を追加する
3. 大きなファイルをより効率的にコピーするためのバッファサイズの最適化
4. 既存のファイルが存在する場合に確認プロンプトを表示する機能を追加する
5. ディレクトリ全体を再帰的にコピーする機能を実装する
6. メモリマップトファイル（mmap）を使用したバージョンを実装して性能比較を行う 
