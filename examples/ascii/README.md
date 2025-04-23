# ASCII サンプル解説

このサンプルは『Programming Rust』第2版の第22章「Unsafe Code」で紹介されているASCIIテキスト処理の実装例です。安全なRustコードとunsafeコードの適切な使い分けを示しています。

## 概要

このモジュール（`my_ascii`）は、ASCII文字のみを含むことが保証された文字列型（`Ascii`）を提供します。主な機能：

- ASCIIのみを含むデータを検証して保持
- 安全なコンストラクタと、パフォーマンス向上のためのunsafeコンストラクタ
- Stringへの効率的な変換

## 重要なコンポーネント

### `Ascii` 構造体

```rust
#[derive(Debug, Eq, PartialEq)]
pub struct Ascii(
    // ASCIIテキストだけを保持することが保証されている：
    // 0から0x7fまでのバイト
    Vec<u8>
);
```

これはタプル構造体で、内部的には単なるバイトのベクトル（`Vec<u8>`）ですが、常にASCII文字（0x00〜0x7F）のみを含むという不変条件を持ちます。

### 安全なコンストラクタ

```rust
pub fn from_bytes(bytes: Vec<u8>) -> Result<Ascii, NotAsciiError> {
    if bytes.iter().any(|&byte| !byte.is_ascii()) {
        return Err(NotAsciiError(bytes));
    }
    Ok(Ascii(bytes))
}
```

このメソッドはバイト列を受け取り、すべてのバイトがASCII範囲内にあるか検証します。非ASCIIバイトがあると`NotAsciiError`を返します。

### 安全でないコンストラクタ

```rust
pub unsafe fn from_bytes_unchecked(bytes: Vec<u8>) -> Ascii {
    Ascii(bytes)
}
```

このメソッドは検証をスキップして直接`Ascii`値を構築します。パフォーマンスが向上しますが、呼び出し側がバイトが実際にASCII範囲内であることを保証する責任を負います。

### 文字列への変換

```rust
impl From<Ascii> for String {
    fn from(ascii: Ascii) -> String {
        // このモジュールにバグがなければ、これは安全です。
        // なぜなら、適切に形成されたASCIIテキストは
        // 適切に形成されたUTF-8でもあるからです。
        unsafe { String::from_utf8_unchecked(ascii.0) }
    }
}
```

`Ascii`から`String`への変換は、通常のUTF-8検証をスキップして効率的に行われます。これは、ASCIIがUTF-8の部分集合であるため安全です（`Ascii`型の不変条件が維持されている限り）。

## テストケース

### 正常系テスト: `good_ascii`

```rust
#[test]
fn good_ascii() {
    use my_ascii::Ascii;
    
    let bytes: Vec<u8> = b"ASCII and ye shall receive".to_vec();
    
    // このコールはメモリ割り当てやテキストコピーではなく、単なるスキャンです
    let ascii: Ascii = Ascii::from_bytes(bytes).unwrap();
    
    // このコールはゼロコスト：メモリ割り当て、コピー、スキャンなし
    let string = String::from(ascii);
    
    assert_eq!(string, "ASCII and ye shall receive");
}
```

正しいASCII文字列の処理方法を示しています。

### 異常系テスト: `bad_ascii`

```rust
#[test]
fn bad_ascii() {
    use my_ascii::Ascii;
    
    // 複雑なプロセスの結果としてこのベクトルがASCIIになると
    // 期待していたと想像してください。何かがうまくいかなかった！
    let bytes = vec![0xf7, 0xbf, 0xbf, 0xbf];
    
    let ascii = unsafe {
        // このunsafe関数の契約はbytesが非ASCIIバイトを
        // 持つときに違反します
        Ascii::from_bytes_unchecked(bytes)
    };
    
    let bogus: String = ascii.into();
    
    // bogusは不正なUTF-8を保持します。最初の文字を解析すると
    // 有効なUnicodeコードポイントではないcharが生成されます。
    // これは未定義の動作なので、言語はこのアサーションがどのように
    // 振る舞うべきかを規定していません。
    assert_eq!(bogus.chars().next().unwrap() as u32, 0x1fffff);
}
```

このテストは、unsafeコードの誤用がどのように未定義動作につながるかを意図的に示しています。教育目的で含まれていますが、実際のコードではこのような状況は避けるべきです。

## 学習ポイント

1. **安全な抽象化**: `Ascii`型は、内部的には単なるバイト列ですが、APIを通じて型安全性を提供しています。

2. **unsafe使用の原則**:
   - unsafeブロックは最小限に保つ
   - 明確に文書化された契約を持つ
   - 安全なAPIで包む

3. **パフォーマンス最適化**: 
   - 安全性を犠牲にせずに不要なチェックを省略
   - 実行時コストとメモリ使用量のバランスを取る

4. **エラー処理**: 
   - 無効な入力に対して適切なエラー型を提供
   - クライアントに元のデータを返して回復を可能にする

## 実験のアイデア

このサンプルをベースに、以下のような実験を試してみてください：

1. `from_bytes`メソッドの代替実装（例：ループではなく別の方法でASCIIをチェック）
2. `Ascii`型に追加のメソッド（例：長さの取得、特定の文字の検索）の実装
3. 安全なAPIを使ったunsafeコンストラクタの使用例
4. パフォーマンステストを追加して安全な実装とunsafe実装を比較 
