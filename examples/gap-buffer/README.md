# ギャップバッファサンプル

このサンプルは『Programming Rust』第2版の第22章「Unsafeコード」の「生ポインタ」セクションで紹介されている、テキストエディタでよく使われるギャップバッファのデータ構造の実装例です。

## 概要

ギャップバッファは、テキストエディタなどでテキストの挿入や削除が頻繁に行われる場所の近くにギャップ（空き領域）を維持することで、編集操作を効率的に行えるようにするデータ構造です。このサンプルでは、低レベルなメモリ操作を行うために`unsafe`コードを使用して、効率的なギャップバッファを実装しています。

## 学べる概念

- 生ポインタ（raw pointer）とその操作
- メモリレイアウトとアドレス計算
- `unsafe`ブロックと関数の適切な使用
- ポインタ演算
- `std::ptr::read`と`std::ptr::write`の使用
- メモリの安全な管理と抽象化
- パフォーマンスを考慮したデータ構造の設計

## 主要コンポーネント

### `GapBuffer<T>` 構造体

ギャップバッファを表す汎用的な構造体です。

```rust
pub struct GapBuffer<T> {
    // 全要素（ギャップを含む）の格納領域
    storage: Vec<MaybeUninit<T>>,
    
    // ギャップの先頭インデックス
    gap_start: usize,
    
    // ギャップの長さ
    gap_len: usize
}
```

### 主要メソッド

#### `new` - 新しいギャップバッファの作成

```rust
pub fn new() -> GapBuffer<T> {
    GapBuffer {
        storage: Vec::new(),
        gap_start: 0,
        gap_len: 0
    }
}
```

#### `insert` - 要素の挿入

```rust
pub fn insert(&mut self, index: usize, elem: T) {
    // ギャップをインデックス位置に移動
    self.move_gap(index);
    
    // ギャップが空であれば拡張
    if self.gap_len == 0 {
        self.enlarge_gap();
    }
    
    // 要素をギャップの先頭に挿入
    unsafe {
        std::ptr::write(self.gap_start_ptr(), elem);
    }
    
    // ギャップの開始位置とサイズを更新
    self.gap_start += 1;
    self.gap_len -= 1;
}
```

#### `remove` - 要素の削除

```rust
pub fn remove(&mut self, index: usize) -> T {
    // インデックスがギャップの直後を指している場合
    if index == self.gap_start {
        // ギャップを1つ右に拡張（=要素を削除）
        self.gap_len += 1;
        // 削除した要素を返す
        unsafe {
            std::ptr::read(self.gap_end_ptr().sub(1))
        }
    } else {
        // ギャップをインデックス位置に移動
        self.move_gap(index);
        // 要素を読み取り、ギャップを1つ右に拡張
        self.gap_len += 1;
        unsafe {
            std::ptr::read(self.gap_start_ptr().sub(1))
        }
    }
}
```

## 使用方法

```rust
use gap_buffer::GapBuffer;

// 新しいギャップバッファを作成
let mut buffer = GapBuffer::new();

// 要素を挿入
buffer.insert(0, 'H');
buffer.insert(1, 'e');
buffer.insert(2, 'l');
buffer.insert(3, 'l');
buffer.insert(4, 'o');

// 要素にアクセス
assert_eq!(buffer[0], 'H');
assert_eq!(buffer[4], 'o');

// 要素を削除
let c = buffer.remove(1); // 'e'を削除
assert_eq!(c, 'e');
assert_eq!(buffer.len(), 4);

// イテレート
let chars: Vec<char> = buffer.iter().collect();
assert_eq!(chars, vec!['H', 'l', 'l', 'o']);
```

## 学習ポイント

1. **生ポインタの安全な使用**: `unsafe`ブロック内で生ポインタを使用していますが、外部からはそれを安全に抽象化しています。

2. **メモリレイアウト**: ギャップバッファの内部メモリレイアウトと、ギャップの移動方法を学べます。

3. **ポインタ演算**: ポインタの加算、減算、およびオフセット計算を行う方法を示しています。

4. **`std::ptr`関数**: `read`や`write`などの低レベルなメモリ操作関数を使用して、要素の移動や配置を行っています。

5. **型の初期化とドロップ**: `MaybeUninit<T>`を使用して、初期化されていない領域を安全に扱う方法を示しています。

6. **アモタイズドコスト**: ギャップを拡張するコストが高いですが、そのコストを多数の操作に分散させることで、平均的なコストを低減しています。

## 発展課題

1. ギャップバッファをテキストエディタの実装に統合する
2. ギャップサイズの調整アルゴリズムを改良して、性能をさらに向上させる
3. 複数のギャップをサポートするように拡張する
4. ギャップバッファのパフォーマンスを他のデータ構造（例：ロープ、ピースチェーン）と比較する
5. メモリ使用量を最適化するためのコンパクト化機能を追加する
6. 変更の履歴と元に戻す/やり直す機能をサポートする
7. マルチスレッド対応の並行ギャップバッファを実装する 
