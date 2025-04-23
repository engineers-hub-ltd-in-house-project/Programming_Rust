# spawn-blocking サンプル

このサンプルは『Programming Rust』第2版の第20章「非同期プログラミング」で紹介されている、非同期コードの中でブロッキング処理を実行するための`spawn_blocking`関数の実装例です。

## 概要

このプログラムは、非同期プログラミングにおける一般的な課題である「ブロッキング操作をどのように扱うか」に対する解決策を示しています。非同期ランタイムのイベントループをブロックせずに、CPU負荷の高い処理や同期I/O操作を別のスレッドで実行する機能を提供します。

## 学べる概念

- 非同期プログラミングとブロッキング処理の連携
- スレッドプールを使った並行処理
- チャネルを使った非同期通信
- `Future`の手動実装
- 非同期ランタイムのパフォーマンスとスケーラビリティ
- 共有状態の安全な管理

## 主要コンポーネント

### `spawn_blocking`関数

ブロッキング処理を別スレッドで実行するための関数です。

```rust
pub fn spawn_blocking<F, R>(f: F) -> impl Future<Output = R>
where
    F: FnOnce() -> R + Send + 'static,
    R: Send + 'static,
{
    // チャネルを作成して結果を送受信
    let (sender, receiver) = oneshot::channel();
    
    // 専用のスレッドプールでブロッキング処理を実行
    THREAD_POOL.execute(move || {
        // クロージャを実行し結果を送信
        let _ = sender.send(f());
    });
    
    // 結果を待機するFutureを返す
    async move {
        receiver.await.expect("ブロッキングタスクはパニックしました")
    }
}
```

### グローバルスレッドプール

ブロッキング処理を実行するためのスレッドプールを管理します。

```rust
lazy_static! {
    static ref THREAD_POOL: ThreadPool = {
        // 論理CPUコア数に基づいてスレッド数を決定
        let num_threads = std::thread::available_parallelism()
            .map(|n| n.get())
            .unwrap_or(4);
        
        // スレッドプールを作成
        ThreadPoolBuilder::new()
            .num_threads(num_threads)
            .thread_name(|i| format!("blocking-{}", i))
            .build()
            .expect("スレッドプールの作成に失敗しました")
    };
}
```

### 使用例

非同期処理の中でCPU負荷の高い処理を行う例です。

```rust
async fn process_data(data: Vec<u32>) -> u32 {
    // 重い計算処理を別スレッドで実行
    let sum = spawn_blocking(move || {
        println!("別スレッドで計算中: {:?}", std::thread::current().name());
        data.iter().sum::<u32>()
    }).await;
    
    println!("計算結果: {}", sum);
    sum
}
```

## 実行方法

```bash
# ビルド
cargo build

# 実行
cargo run

# または直接バイナリを実行
./target/debug/spawn-blocking
```

## 学習ポイント

1. **非同期とブロッキングの共存**: 非同期ランタイムを止めることなく、ブロッキング処理を実行する方法を学べます。

2. **スレッドプールの管理**: 適切なスレッド数のプールを作成し、効率的に作業を分散する方法を示しています。

3. **チャネルによる通信**: ワンショットチャネルを使用して、非同期コードとブロッキングコード間で結果を伝達する方法を学べます。

4. **Future実装**: 非同期処理の結果を表す`Future`を手動で実装する方法を示しています。

5. **リソース管理**: スレッドの生成とリソース使用量のバランスをとる方法を学べます。

## 発展課題

1. スレッドプールの設定を動的に変更できるようにする
2. キャンセレーション機能を追加して、長時間実行中のブロッキングタスクを中断できるようにする
3. ブロッキングタスクの優先順位付けとスケジューリング機能を実装する
4. タスクのキューイングと最大同時実行数の制限を追加する
5. パフォーマンス指標（実行時間、スレッド使用率など）を収集して表示する機能を追加する
6. 実際のユースケース（画像処理、ファイルI/Oなど）でパフォーマンスをテストする 
