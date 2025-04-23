# 基本ルーターサンプル

このサンプルは『Programming Rust』第2版の第20章「非同期プログラミング」で紹介されている、シンプルなHTTPルーターの実装例です。

## 概要

このライブラリは、非同期APIを持つシンプルなHTTPルーターを提供します。URLパスとHTTPメソッドに基づいてリクエストを適切なハンドラ関数にルーティングする機能を持ち、Webフレームワークの基本的な要素を示しています。

## 学べる概念

- 非同期関数と`async`/`await`の使用法
- トレイトオブジェクトによる動的ディスパッチ
- クロージャと高階関数の活用
- `Box<dyn Future>`によるヘテロジニアスな非同期処理
- エラー処理とResult型の使用
- ジェネリックプログラミングとトレイト境界
- ビルダーパターンの実装

## 主要コンポーネント

### `Router` 構造体

URLパスとHTTPメソッドに基づいてリクエストを適切なハンドラにルーティングする中心的なコンポーネントです。

```rust
pub struct Router<C> {
    // ルートのマップ: メソッド + パス -> ハンドラ関数
    routes: HashMap<(Method, String), BoxHandler<C>>,
    
    // 見つからなかった時のハンドラ
    not_found: BoxHandler<C>,
}
```

### `Handler` トレイト

リクエストを処理するハンドラ関数を表す非同期トレイトです。

```rust
pub trait Handler<C>: Send + Sync + 'static {
    fn handle(&self, context: C, request: Request) -> BoxFuture<'static, Response>;
}
```

### `BoxHandler<C>` 型

型消去されたハンドラ型で、異なる型のハンドラを単一のコレクションで管理できるようにします。

```rust
pub type BoxHandler<C> = Box<dyn Handler<C>>;
```

## 使用方法

```rust
use basic_router::{Request, Response, Router, Method};
use std::collections::HashMap;

// コンテキスト型（この例では単純な文字列）
type Context = String;

// ルーターの作成
let mut router = Router::<Context>::new();

// ルートの登録
router.add_route(Method::GET, "/hello", |_ctx, _req| {
    Box::pin(async {
        Response {
            status: 200,
            body: "Hello, World!".to_string(),
            ..Response::default()
        }
    })
});

// パラメータを受け取るルートの登録
router.add_route(Method::GET, "/users/:id", |_ctx, req| {
    let id = req.params.get("id").cloned().unwrap_or_default();
    Box::pin(async move {
        Response {
            status: 200,
            body: format!("User ID: {}", id),
            ..Response::default()
        }
    })
});

// リクエストの処理
let context = "app_context".to_string();
let request = Request {
    method: Method::GET,
    path: "/hello".to_string(),
    params: HashMap::new(),
    body: "".to_string(),
};

// 非同期でリクエストを処理
async_std::task::block_on(async {
    let response = router.route(context, request).await;
    assert_eq!(response.status, 200);
    assert_eq!(response.body, "Hello, World!");
});
```

## 学習ポイント

1. **非同期プログラミングの基礎**: `async`/`await`構文を使用して、ノンブロッキングなコードを記述する方法を示しています。

2. **トレイトオブジェクトと型消去**: `dyn Handler`と`Box<dyn Future>`を使用して、異なる型のハンドラと非同期処理を単一のインターフェースで操作しています。

3. **クロージャ**: ルートハンドラとしてクロージャを使用して、コードの簡潔さと柔軟性を高めています。

4. **パラメータ抽出**: URLパスからパラメータを抽出する方法を示しています。

5. **エラー処理**: 適切なエラー処理とフォールバックメカニズム（not_found ハンドラ）の実装方法を示しています。

## 発展課題

1. ミドルウェアのサポートを追加する（例：ロギング、認証、CORS処理など）
2. 正規表現ベースのルーティングを実装する
3. パスパラメータをより柔軟に抽出する機能を追加する
4. テンプレートエンジンとの統合を行う
5. JSONやフォームデータなどの自動的なリクエスト本文解析を実装する
6. レスポンスキャッシュ機能を追加する
7. 非同期データベース接続との統合例を作成する 
