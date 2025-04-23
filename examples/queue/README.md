# キュー実装サンプル

このサンプルは『Programming Rust』第2版の第9章「構造体」で紹介されている`char`型のキューを実装したライブラリです。

## 概要

このライブラリは、FIFO（先入れ先出し）原則に従って`char`型の値を格納するキューデータ構造を実装しています。キューは内部的に`Vec<char>`を使用して実装されています。

## 学べる概念

- 構造体の定義と実装
- メソッドの追加
- 内部状態の管理
- 要素の追加と取り出し
- Vecなどの標準コレクションの利用
- モジュール化とライブラリの作成

## 主要コンポーネント

### `Queue` 構造体

文字のキューを表す構造体です。

```rust
pub struct Queue {
    older: Vec<char>,  // 古い要素（先に取り出される）
    younger: Vec<char> // 新しい要素（後で取り出される）
}
```

### 主要メソッド

#### `new` - キューの作成

```rust
pub fn new() -> Queue {
    Queue { older: Vec::new(), younger: Vec::new() }
}
```

#### `push` - 要素の追加

```rust
pub fn push(&mut self, c: char) {
    self.younger.push(c);
}
```

#### `pop` - 要素の取り出し

```rust
pub fn pop(&mut self) -> Option<char> {
    if self.older.is_empty() {
        if self.younger.is_empty() {
            return None;
        }
        
        // 新しい要素を古い要素の位置に移動（順序を逆転させる）
        self.older = self.younger.drain(..).rev().collect();
    }
    
    self.older.pop()
}
```

#### `is_empty` - キューが空かどうかの確認

```rust
pub fn is_empty(&self) -> bool {
    self.older.is_empty() && self.younger.is_empty()
}
```

## 使用方法

```rust
use queue::Queue;

// 新しいキューを作成
let mut q = Queue::new();

// 要素を追加
q.push('A');
q.push('B');
q.push('C');

// 要素を取り出す（FIFOなのでAが最初に出てくる）
assert_eq!(q.pop(), Some('A'));
assert_eq!(q.pop(), Some('B'));

// さらに要素を追加
q.push('D');

// 残りの要素を取り出す
assert_eq!(q.pop(), Some('C'));
assert_eq!(q.pop(), Some('D'));
assert_eq!(q.pop(), None);  // 空になったのでNoneが返る
```

## 学習ポイント

1. **効率的なデータ構造**: 2つのベクトルを使用することで、要素の追加と削除の両方を効率的に行う方法を示しています。

2. **アモタイズドコスト**: `pop`操作は時々コストが高い（`younger`から`older`へのデータ移動が必要）ですが、平均的には定数時間で実行できます。

3. **不変条件の管理**: `pop`操作での要素の再配置によって、キューの正しい動作（FIFO順序）を保証しています。

4. **カプセル化**: 内部実装の詳細（2つのベクトルの使用）を隠蔽し、シンプルなAPIを提供しています。

5. **Option型の使用**: 空のキューからの取り出し操作を安全に扱うために`Option`型を使用しています。

## 発展課題

1. このキューを任意の型T用のジェネリックなキューに拡張する（generic-queueサンプルを参照）
2. イテレータを実装して、キューの要素を消費せずに走査できるようにする
3. キャパシティ（容量）の制限付きキューを実装する
4. デバッグ出力のためのDisplayトレイトを実装する
5. キューの状態をシリアライズ/デシリアライズする機能を追加する 
