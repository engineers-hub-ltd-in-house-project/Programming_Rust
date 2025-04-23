# libgit2 Rust バインディングサンプル

このサンプルは『Programming Rust』第2版の第21章「異言語関数インターフェース」で紹介されている、C言語のlibgit2ライブラリに対するRustの低レベルバインディングの実装例です。

## 概要

このプロジェクトでは、Rustの外部関数インターフェース（FFI）機能を使用して、C言語で書かれた人気のGitライブラリである`libgit2`にアクセスする方法を示しています。低レベルなCのAPIをRustから直接呼び出す例として実装されています。

## 学べる概念

- 外部関数インターフェース（FFI）の基礎
- `extern "C"`ブロックによるC言語関数の宣言
- C言語の型とRustの型のマッピング
- ポインタと生ポインタの安全な扱い方
- `build.rs`を使用したビルド設定
- `unsafe`ブロックの適切な使用
- システムライブラリとの連携

## 主要コンポーネント

### `raw.rs`

C言語のlibgit2 APIを直接Rustから呼び出すための低レベルバインディングを定義しています。

```rust
// libgit2のC言語APIを表現するextern "C"ブロック
#[link(name = "git2")]
extern "C" {
    pub fn git_repository_open(out: *mut *mut git_repository, path: *const c_char) -> c_int;
    pub fn git_repository_free(repo: *mut git_repository);
    // 他のlibgit2関数...
}

// libgit2の型をRustで表現
pub enum git_repository {}
pub enum git_commit {}
// その他の必要な型...
```

### `main.rs`

libgit2バインディングを使用してGitリポジトリにアクセスする例を示しています。

```rust
fn main() {
    // コマンドライン引数からリポジトリパスを取得
    let path = std::env::args().nth(1)
        .expect("usage: libgit2-rs PATH");
    
    // libgit2を使用してリポジトリを開く
    unsafe {
        let repo = open_repository(&path);
        // リポジトリ情報を表示
        // ...
        // リソース解放
        raw::git_repository_free(repo);
    }
}
```

### `build.rs`

ビルド時にlibgit2ライブラリをリンクするための設定を行います。

```rust
fn main() {
    // libgit2ライブラリへのリンク指定
    println!("cargo:rustc-link-lib=git2");
    // 必要に応じてlibgit2ライブラリの場所を指定
    // println!("cargo:rustc-link-search=/path/to/libgit2");
}
```

## 実行方法

```bash
# システムにlibgit2-devをインストール（Ubuntuの場合）
sudo apt-get install libgit2-dev

# ビルド
cargo build

# 実行（リポジトリへのパスを指定）
cargo run -- /path/to/git/repository

# または直接バイナリを実行
./target/debug/libgit2-rs /path/to/git/repository
```

## 学習ポイント

1. **FFIの基本**: Rustから外部のCライブラリを呼び出す方法の基本を学べます。

2. **型のマッピング**: CとRustの間で型をマッピングする方法を示しています。特に、構造体やポインタの扱いに注目してください。

3. **メモリ管理**: Cライブラリから取得したリソースを適切に解放する方法を学べます。

4. **`unsafe`の使用**: FFIコードは本質的に安全でないため、`unsafe`ブロックを使用して明示的に示す方法を学べます。

5. **ビルドスクリプト**: `build.rs`を使用して外部ライブラリをRustプロジェクトにリンクする方法を示しています。

## 発展課題

1. このローレベルバインディングの上に安全な高レベルAPIを構築する（libgit2-rs-safeサンプル参照）
2. エラー処理を改善し、libgit2のエラーコードをRustのエラー型に変換する
3. メモリリークを防ぐためのRAIIパターンを実装する
4. 非同期APIをサポートするために`tokio`などと統合する
5. GitHubのrust-lang/git2-rsクレートと比較して、設計の違いを学ぶ 
