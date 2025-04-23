# シダ植物シミュレーター

細胞レベルからシダ植物の成長をシミュレートするRustライブラリです。

## 機能

- 様々なシダ植物タイプ（巻きシダなど）のシミュレーション
- 茎や葉などの植物構造のモデリング
- 胞子生成などの生物学的プロセスのシミュレーション
- 制御された成長実験のためのテラリウム環境
- シミュレーション共有のためのネットワーク機能

## 使用例

```rust
use fern_sim::{Terrarium, connect};
use std::time::Duration;

// 新しいテラリウムを作成
let mut terrarium = Terrarium::new();

// またはファイルから読み込み
let mut terrarium = Terrarium::load("my_terrarium.tm");

// 日光を与えてシミュレーション実行
terrarium.apply_sunlight(Duration::from_secs(60));

// テラリウム内のシダにアクセス
let fern = terrarium.fern(0);
if !fern.is_furled() {
    println!("シダの葉が開きました！");
}

// オンラインギャラリーにアップロード
let mut session = connect();
session.upload_all();
```

## プロジェクト構造

- `plant_structures`：シダの構成要素（茎、葉など）のモデル
- `simulation`：コアシミュレーションロジック
- `spores`：シダの繁殖シミュレーション
- `net`：シミュレーション共有用のネットワーク機能

## ライセンス

このサンプルは「プログラミングRust」書籍のサンプルリポジトリの一部です。 
