# JSON マクロサンプル

このサンプルは『Programming Rust』第2版の第21章「マクロ」で紹介されている、宣言的マクロを使用してJSONデータを構築する例です。

## 概要

このライブラリでは、Rustのマクロシステムを使用して、JSONデータを簡潔に構築するための`json!`マクロを実装しています。このマクロにより、JavaScriptのオブジェクトリテラルに似た構文でJSON値を簡単に作成できます。

## 学べる概念

- 宣言的マクロ（`macro_rules!`）の定義と使用
- マクロのパターンマッチング
- 再帰的なマクロ展開
- マクロの衛生性（hygiene）
- 式とステートメントの扱い
- JSONのデータ構造

## 主要コンポーネント

### `json!`マクロ

JSONデータを簡潔に構築するためのマクロです。

```rust
macro_rules! json {
    // null
    (null) => {
        Json::Null
    };
    
    // 配列
    ([ $( $element:tt ),* ]) => {
        Json::Array(vec![ $( json!($element) ),* ])
    };
    
    // オブジェクト
    ({ $( $key:tt : $value:tt ),* }) => {
        {
            let mut fields = Box::new(std::collections::HashMap::new());
            $( fields.insert($key.to_string(), json!($value)); )*
            Json::Object(fields)
        }
    };
    
    // その他のリテラル（数値、文字列、真偽値）
    ($other:tt) => {
        Json::from($other)
    };
}
```

### `Json`列挙型

JSONの値を表現するための列挙型です。

```rust
#[derive(Clone, PartialEq, Debug)]
pub enum Json {
    Null,
    Boolean(bool),
    Number(f64),
    String(String),
    Array(Vec<Json>),
    Object(Box<HashMap<String, Json>>)
}
```

## 使用方法

```rust
// 基本的な値
let null = json!(null);
let boolean = json!(true);
let number = json!(42);
let string = json!("hello");

// 配列
let array = json!([1, 2, 3, 4]);

// ネストされたオブジェクト
let object = json!({
    "name": "John Doe",
    "age": 30,
    "address": {
        "street": "123 Main St",
        "city": "Anytown"
    },
    "phones": [
        "+1 555-1234",
        "+1 555-5678"
    ]
});

println!("{:#?}", object);
```

## 学習ポイント

1. **マクロの構文**: `macro_rules!`を使用した宣言的マクロの定義方法を学べます。

2. **パターンマッチング**: マクロがさまざまな入力形式（null, 配列, オブジェクト, リテラル）をどのように区別するかを示しています。

3. **再帰的マクロ**: 配列要素やオブジェクトの値に対して自分自身を再帰的に呼び出す方法を示しています。

4. **繰り返しパターン**: `$( ... ),*` のような繰り返しパターンを使用して、可変長の引数リストを処理する方法を学べます。

5. **トークンツリー**: `tt` 指定子を使用してマクロ引数をトークンツリーとして捕捉する方法を示しています。

## 発展課題

1. JSONのプリティプリント機能を追加する
2. JSONの検証機能を追加する（スキーマバリデーションなど）
3. JSONからRustの構造体への変換機能を実装する（デシリアライズ）
4. マクロを拡張して、既存のJSONにフィールドを追加する構文をサポートする
5. 型安全なJSONビルダーAPIを作成し、マクロでそれを呼び出す
6. プロシージャルマクロ（derive マクロなど）を使用したバージョンを実装する 
