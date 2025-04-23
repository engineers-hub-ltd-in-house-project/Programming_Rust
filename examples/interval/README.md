# 区間型サンプル

このサンプルは『Programming Rust』第2版の第12章「演算子オーバーロード」で紹介されている区間（インターバル）型と比較演算子の実装例です。

## 概要

このライブラリでは、数値の区間を表す`Interval<T>`型を実装しています。区間は始点と終点を持ち、それらの間の値を表現します。さらに、区間同士の半順序関係を定義するために`PartialOrd`トレイトを実装しています。

## 学べる概念

- ジェネリック型の定義
- 比較演算子のオーバーロード
- `PartialOrd`トレイトの実装
- 半順序関係の概念
- デバッグと表示用のトレイト
- トレイト境界の使い方

## 主要コンポーネント

### `Interval<T>` 構造体

範囲を表すジェネリック構造体です。型パラメータ`T`は順序付け可能な型（数値型など）を想定しています。

```rust
#[derive(Copy, Clone, Debug)]
pub struct Interval<T> {
    pub start: T,
    pub end: T
}
```

### `PartialOrd`の実装

区間の半順序関係を定義するために`PartialOrd`トレイトを実装しています。区間`a`が区間`b`よりも小さいとは、`a`の終点が`b`の始点以下であることを意味します。

```rust
impl<T: PartialOrd> PartialOrd for Interval<T> {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        if self == other {
            Some(Ordering::Equal)
        } else if self.end <= other.start {
            Some(Ordering::Less)
        } else if self.start >= other.end {
            Some(Ordering::Greater)
        } else {
            None
        }
    }
}
```

### `PartialEq`の実装

区間の等価性を判定するために`PartialEq`トレイトを実装しています。

```rust
impl<T: PartialEq> PartialEq for Interval<T> {
    fn eq(&self, other: &Self) -> bool {
        self.start == other.start && self.end == other.end
    }
}
```

## 使用方法

```rust
use interval::Interval;
use std::cmp::Ordering;

// 区間の作成
let a = Interval { start: 10, end: 20 };
let b = Interval { start: 20, end: 30 };
let c = Interval { start: 15, end: 25 };

// 比較演算子を使った比較
assert!(a < b);         // a の終点 <= b の始点
assert!(!(a < c));      // a と c は重なっているため比較不能
assert!(!(a >= c));     // 同様に比較不能

// partial_cmp で直接比較
assert_eq!(a.partial_cmp(&b), Some(Ordering::Less));
assert_eq!(a.partial_cmp(&c), None);  // 比較不能なのでNone
```

## 学習ポイント

1. **半順序関係**: すべての値が比較可能とは限らない関係を表現する方法を学べます。例えば、区間同士が重なる場合、どちらが「大きい」かは定義できません。

2. **`partial_cmp`メソッド**: `PartialOrd`トレイトの核となる`partial_cmp`メソッドを実装する方法を学べます。このメソッドは比較可能な場合に順序を`Option<Ordering>`として返し、比較不能な場合に`None`を返します。

3. **ジェネリックコード**: 型パラメータ`T`とトレイト境界`T: PartialOrd`を使用して、様々な数値型に対応する汎用的な実装を作成しています。

4. **演算子オーバーロード**: `PartialOrd`トレイトを実装することで、`<`、`<=`、`>`、`>=`などの比較演算子を使用できるようになります。

5. **Option型の活用**: 半順序関係における「比較不能」の状態を`Option::None`を使って表現しています。

## 発展課題

1. 区間の重なりを判定するメソッド`overlaps`を追加する
2. 二つの区間の和集合（ユニオン）と積集合（インターセクション）を計算するメソッドを実装する
3. 区間のリストを受け取り、重なりのない区間のリストに変換する関数を実装する
4. 浮動小数点数の特殊値（NaNなど）を考慮した実装に拡張する
5. 区間の集合演算（補集合、対称差など）を実装する
6. 多次元の区間（長方形や直方体など）に拡張する 
