# libgit2 安全なRustラッパーサンプル

このサンプルは『Programming Rust』第2版の第21章「異言語関数インターフェース」で紹介されている、C言語のlibgit2ライブラリに対する安全なRustラッパーの実装例です。

## 概要

このプロジェクトでは、C言語で書かれたGitライブラリ`libgit2`に対して、低レベルなFFIバインディングの上に安全で使いやすい高レベルAPIを構築する方法を示しています。Rustの型システムとライフタイムを活用して、メモリ安全性とリソース管理を確保しながらC言語ライブラリを扱う実践的な例となっています。

## 学べる概念

- 安全なRustラッパーの設計パターン
- RAIIによるリソース管理
- モジュール構造とAPI設計
- エラー処理とResult型の活用
- ライフタイムによる参照の安全性確保
- `unsafe`コードの隔離と抽象化
- 型システムを活用した安全性の強制

## 主要コンポーネント

### `git/raw.rs`

低レベルなlibgit2のFFIバインディングを定義しています。これは`libgit2-rs`サンプルと似ていますが、安全なAPIの基盤となります。

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

### `git/mod.rs`

安全なRust APIを定義し、低レベルな`raw`モジュールの上に構築しています。

```rust
use std::path::Path;
use std::ffi::CString;
use std::ptr;

mod raw;

pub struct Repository {
    // libgit2のリポジトリへの生ポインタをカプセル化
    raw: *mut raw::git_repository
}

impl Repository {
    /// 指定されたパスにあるGitリポジトリを開きます
    pub fn open<P: AsRef<Path>>(path: P) -> Result<Repository, GitError> {
        // パスをCStringに変換
        let path_str = path.as_ref().to_str().ok_or(GitError::InvalidPath)?;
        let c_path = CString::new(path_str).map_err(|_| GitError::InvalidPath)?;
        
        unsafe {
            let mut repo = ptr::null_mut();
            let result = raw::git_repository_open(&mut repo, c_path.as_ptr());
            if result < 0 {
                return Err(GitError::from_code(result));
            }
            Ok(Repository { raw: repo })
        }
    }
    
    // リポジトリに対する安全な操作メソッド...
}

// RAIIパターンによるリソース管理
impl Drop for Repository {
    fn drop(&mut self) {
        unsafe {
            raw::git_repository_free(self.raw);
        }
    }
}

// エラー型
#[derive(Debug)]
pub enum GitError {
    InvalidPath,
    NotFound,
    Generic(i32),
    // その他のエラー...
}

impl GitError {
    fn from_code(code: i32) -> Self {
        // libgit2のエラーコードをRustのエラー型に変換
        match code {
            -3 => GitError::NotFound,
            _ => GitError::Generic(code),
        }
    }
}
```

### `main.rs`

安全なAPIを使用してGitリポジトリにアクセスするサンプルコードです。

```rust
use std::env;
use libgit2_rs_safe::git::Repository;

fn main() {
    // コマンドライン引数からリポジトリパスを取得
    let path = env::args().nth(1).expect("usage: libgit2-rs-safe PATH");
    
    // 安全なAPIを使用してリポジトリを開く
    match Repository::open(&path) {
        Ok(repo) => {
            println!("リポジトリを開きました: {}", path);
            // リポジトリに対する操作...
            // Dropトレイトによってリポジトリは自動的に解放される
        }
        Err(err) => {
            eprintln!("リポジトリを開けませんでした: {:?}", err);
        }
    }
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
./target/debug/libgit2-rs-safe /path/to/git/repository
```

## 学習ポイント

1. **安全な抽象化**: 低レベルな`unsafe`コードを安全なRust APIで抽象化する方法を学べます。

2. **RAIIによるリソース管理**: `Drop`トレイトを実装して、オブジェクトが破棄されるときに自動的にリソースを解放する方法を示しています。

3. **エラー処理**: CのAPIからのエラーコードを意味のあるRustのエラー型に変換する方法を学べます。

4. **モジュール設計**: 低レベルな`raw`モジュールを内部に隠し、安全なAPIだけを公開する設計パターンを示しています。

5. **型安全性**: Rustの型システムを活用して、APIの安全な使用を強制する方法を学べます。

## 発展課題

1. より多くのlibgit2機能（コミット、ブランチ、タグなど）に対する安全なラッパーを追加する
2. テスト駆動開発（TDD）アプローチを使用して安全なAPIをさらに拡張する
3. ドキュメントコメントを充実させて、ユーザーフレンドリーなAPIドキュメントを生成する
4. 非同期操作をサポートするために`tokio`や`async-std`と統合する
5. 本格的なGitクライアントアプリケーションを構築する基盤として拡張する 
