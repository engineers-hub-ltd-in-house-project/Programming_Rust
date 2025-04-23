# 複素数型サンプル

このサンプルは『Programming Rust』第2版の第12章「演算子オーバーロード」および第17章「文字列とテキスト」で紹介されている複素数型の実装例です。

## 概要

このライブラリは、複素数（実部と虚部からなる数）を表現する`Complex<T>`型を実装しています。加算、減算、乗算などの基本的な演算子をオーバーロードし、複素数を直感的に操作できるようにしています。また、フォーマット出力のためのトレイトも実装しています。

## 学べる概念

- ジェネリック型の作成と使用
- 演算子オーバーロード（`Add`, `Sub`, `Mul`など）
- 標準ライブラリトレイトの実装
- フォーマット用トレイト（`Display`, `Debug`）の実装
- テストの記述方法

## 主要コンポーネント

### `Complex<T>` 構造体

複素数を表す汎用的な構造体です。型パラメータ`T`は浮動小数点数（`f32`, `f64`など）を想定しています。

```rust
#[derive(Copy, Clone, Debug)]
struct Complex<T> {
    re: T,
    im: T
}
```

### 演算子の実装

複素数の加算、減算、乗算などの演算子を実装しています。

```rust
impl<T> Add for Complex<T>
    where T: Add<Output=T>
{
    type Output = Complex<T>;
    fn add(self, rhs: Complex<T>) -> Self::Output {
        Complex { re: self.re + rhs.re, im: self.im + rhs.im }
    }
}
```

### フォーマット実装

複素数を表示するための`Display`トレイトを実装しています。

```rust
impl<T> Display for Complex<T>
    where T: Display
{
    fn fmt(&self, f: &mut Formatter<'_>) -> std::fmt::Result {
        write!(f, "{} + {}i", self.re, self.im)
    }
}
```

## 使用方法

```rust
// 複素数の作成
let a = Complex { re: 1.0, im: 2.0 };
let b = Complex { re: 3.0, im: 4.0 };

// 演算子を使用した計算
let sum = a + b;
let product = a * b;

// 表示
println!("{}", sum);      // "4 + 6i"
println!("{:?}", product); // Complex { re: -5.0, im: 10.0 }
```

## 学習ポイント

1. **演算子オーバーロード**: Rustの標準ライブラリに定義されているトレイト（`Add`, `Sub`, `Mul`など）を実装することで、演算子（`+`, `-`, `*`など）をカスタム型でも使用できます。

2. **ジェネリックプログラミング**: 型パラメータ`T`を使用して、様々な数値型（`f32`, `f64`など）に対応できる汎用的な複素数実装を作成しています。

3. **トレイト境界**: `where T: Add<Output=T>`のような制約を使用して、型パラメータが特定の機能を持つことを要求しています。

4. **フォーマット出力**: `Display`や`Debug`トレイトを実装することで、`println!`などのマクロでカスタム型を表示できます。

5. **カスタムコンストラクタ**: `new`や`from_polar`などのコンストラクタメソッドを実装して、複素数を作成する便利な方法を提供しています。

6. **メソッド実装**: `norm_sqr`や`norm`などのメソッドを実装して、複素数の操作を容易にしています。

## 発展課題

1. 複素数の割り算（`Div`トレイト）を実装する
2. 複素数の等値比較（`PartialEq`トレイト）を実装する
3. 複素数の共役（conjugate）を計算するメソッドを追加する
4. 極形式（polar form）から直交形式（rectangular form）への変換機能を追加する
5. 複素数に関する追加の数学的操作（指数関数、対数関数など）を実装する
6. 複素平面上の可視化機能を追加する 
