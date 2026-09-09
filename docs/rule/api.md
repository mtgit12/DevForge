## APIレスポンスルール

### 設計方針

全エンドポイントで以下の構造に統一する。

```json
// 成功
{
  "status": "success",
  "data": { ... },
}
or
{
  "status": "success",
  "data": true,
}

// エラー
{
  "status": "error",
  "errorMessage": "エラーが発生しました。",
}
```

### エラーコード一覧

| コード | HTTPステータス | 用途 |
|---|---|---|
| `VALIDATION_ERROR` | 422 | バリデーションエラー |
| `DOMAIN_ERROR` | 422 | ビジネスロジックエラー |
| `UNAUTHORIZED` | 401 | 未認証 |
| `FORBIDDEN` | 403 | 権限不足 |
| `NOT_FOUND` | 404 | リソースが存在しない |
| `ROUTE_NOT_FOUND` | 404 | エンドポイントが存在しない |
| `CONFLICT` | 409 | データ競合（重複登録等）|
| `HTTP_ERROR` | 4xx/5xx | HTTP例外（汎用）|
| `SERVER_ERROR` | 500 | サーバー内部エラー |
