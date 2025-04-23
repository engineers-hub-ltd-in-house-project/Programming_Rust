# `block_on` 実装サンプル

このサンプルは『Programming Rust』第2版の第20章「非同期プログラミング」で紹介されている、シンプルな`block_on`関数の実装例です。

## 概要

このサンプルでは、Rustの非同期プログラミングの基盤となる部分を理解するため、`Future`を同期的に実行するための`block_on`関数をスクラッチから実装しています。これにより、非同期ランタイムの基本的な機能の一部がどのように動作するかを学ぶことができます。

## 学べる概念

- Rustの`Future`トレイトとその実装
- ポーリングメカニズム
- `Waker`と`Context`の役割
- ブロッキング実行とノンブロッキング実行の違い
- 非同期ランタイムの基本的な動作原理

## 主要コンポーネント

### `MyWaker`構造体

`Waker`を作成するためのユーティリティ関数を含む構造体です。これは`Future`が完了したとき、または進行可能になったときに通知を受け取るために使用されます。

```rust
struct MyWaker;

fn create_waker() -> Waker {
    // 詳細な実装は省略されていますが、
    // RawWakerVTable と RawWaker を使用して Waker を作成する
    // ...
}
```

### `block_on` 関数

非同期関数を同期的に実行するための関数です。`Future`が完了するまでポーリングし続けます。

```rust
fn block_on<F: Future>(future: F) -> F::Output {
    // Future のピン留めとコンテキストの作成
    pin_mut!(future);
    let waker = create_waker();
    let mut context = Context::from_waker(&waker);
    
    // Future が完了するまでポーリング
    loop {
        match future.as_mut().poll(&mut context) {
            Poll::Ready(val) => break val,
            Poll::Pending => {
                // 何らかの方法で待機（この実装では単純な yield）
                std::thread::yield_now();
            }
        }
    }
}
```

### サンプルの使用例

```rust
async fn example_async_fn() -> u32 {
    // 非同期処理の例
    println!("非同期関数が呼ばれました");
    42
}

fn main() {
    // block_on を使って非同期関数を同期的に実行
    let result = block_on(example_async_fn());
    println!("結果: {}", result);
}
```

## 実行方法

```bash
# ビルド
cargo build

# 実行
cargo run

# または直接バイナリを実行
./target/debug/block-on
```

## 学習ポイント

1. **`Future`の手動ポーリング**: `Future`がどのようにポーリングされ、状態が管理されるかを理解できます。

2. **`Waker`メカニズム**: 非同期タスクの通知システムがどのように機能するかの基本を学べます。

3. **ノンブロッキングとブロッキングの橋渡し**: 非同期コードを同期的に実行する方法とその課題を理解できます。

4. **非同期ランタイムの内部**: トークンリングやウェイクアップメカニズムなど、非同期ランタイムの基本的な構成要素の一部を理解できます。

5. **ゼロからの実装**: 既存のクレートに依存せず、基本的な機能を自分で実装することで、より深い理解が得られます。

## 発展課題

1. `block_on`にタイムアウト機能を追加する
2. より効率的なウェイクアップメカニズムを実装する（条件変数やイベント通知など）
3. 複数の`Future`を同時に実行できるように拡張する
4. エラー処理を追加する
5. ユニットテストを作成して、実装が正しく動作することを確認する
6. 実際の非同期ランタイム（`tokio`や`async-std`など）の`block_on`実装と比較する 
