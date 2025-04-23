# ジェネリックキュー実装サンプル

このサンプルは『Programming Rust』第2版の第9章「構造体」で紹介されているジェネリックなキューを実装したライブラリです。

## 概要

このライブラリは、FIFO（先入れ先出し）原則に従って任意の型`T`の値を格納できるキューデータ構造を実装しています。特定の型に限定されない汎用的なキューとなっており、型パラメータを使用したジェネリックプログラミングの良い例です。

## 学べる概念

- ジェネリック型と型パラメータ
- トレイト境界の使用
- 構造体の実装
- ライフタイムパラメータ
- メソッドの実装
- モジュール化とライブラリの作成

## 主要コンポーネント

### `Queue<T>` 構造体

任意の型Tの値を格納できるキューを表すジェネリック構造体です。

```rust
pub struct Queue<T> {
    older: Vec<T>,   // 古い要素（先に取り出される）
    younger: Vec<T>  // 新しい要素（後で取り出される）
}
```

### 主要メソッド

#### `new` - キューの作成

```rust
pub fn new() -> Queue<T> {
    Queue { older: Vec::new(), younger: Vec::new() }
}
```

#### `push` - 要素の追加

```rust
pub fn push(&mut self, t: T) {
    self.younger.push(t);
}
```

#### `pop` - 要素の取り出し

```rust
pub fn pop(&mut self) -> Option<T> {
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

#### `iter` - イテレータの取得

```rust
pub fn iter(&self) -> Iter<'_, T> {
    Iter { older: self.older.iter(), younger: self.younger.iter().rev() }
}
```

### `IntoIter<T>` と `Iter<'a, T>` 構造体

キューの要素をイテレーションするためのイテレータ実装です。

## 使用方法

```rust
use generic_queue::Queue;

// 整数のキューを作成
let mut q = Queue::new();
q.push(1);
q.push(2);
q.push(3);

// 要素を取り出す
assert_eq!(q.pop(), Some(1));
assert_eq!(q.pop(), Some(2));

// 文字列のキューも同じように作成できる
let mut sq = Queue::new();
sq.push("hello".to_string());
sq.push("world".to_string());

// イテレータを使用した要素へのアクセス
for s in &sq {
    println!("{}", s);
}
```

## 学習ポイント

1. **ジェネリックプログラミング**: 型パラメータ`T`を使用することで、任意の型に対応するキュー実装を提供しています。

2. **トレイト実装**: `IntoIterator`トレイトを実装することで、for-in構文でキューの要素を走査できるようにしています。

3. **ライフタイム**: `Iter`構造体でライフタイムパラメータ`'a`を使用して、キューへの参照が有効である間だけイテレータが使用できることを保証しています。

4. **参照と所有権**: イテレータは要素の参照を提供するため、キューの所有権を移動させることなく要素にアクセスできます。

5. **ドロップによるリソース解放**: Rustの所有権システムにより、キューが不要になったときに自動的にメモリが解放されます。

## 発展課題

1. `Clone`トレイトを実装して、キューのクローンを作成できるようにする
2. `From`や`Into`トレイトを実装して、他のコレクション型（Vec, LinkedList等）との相互変換を可能にする
3. `PartialEq`トレイトを実装して、キュー同士の比較を可能にする
4. スライスや配列からキューを作成するコンストラクタを追加する
5. キューの最大サイズを制限する機能を追加する
6. マルチスレッド対応の機能（Send, Sync）について検討する 
