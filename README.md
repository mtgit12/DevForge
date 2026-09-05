# DevForge

## ディレクトリ構成

```
repository/
├── docker/
│   ├── frontend/
│   │   └── Dockerfile          # Bun + Next.js（dev / builder / runner の多段構成）
│   ├── api/                    # 将来追加
│   └── mysql/
│       ├── conf.d/my.cnf       # MySQL 追加設定
│       └── initdb.d/           # 初回起動時に実行される .sql / .sh
├── frontend/                   # Next.js アプリ本体（create-next-app の出力先）
│   └── .dockerignore
├── api/                        # 将来追加
├── compose.yaml
├── .env.example
├── .gitignore
└── README.md
```

## 前提

- Docker Desktop（Compose v2）
- Next.js 側に `output: "standalone"` の設定が必要

```ts
// frontend/next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  output: "standalone",
};

export default nextConfig;
```

## セットアップ

```bash
# 1. 環境変数を用意
cp .env.example .env

# 2. Next.js アプリを作成（既存プロジェクトがある場合は frontend/ に配置）
bunx create-next-app@latest frontend

# 3. 起動
docker compose up -d --build
```

- フロントエンド: http://localhost:3000
- MySQL: `localhost:3306`（ユーザー / パスワードは `.env` を参照）

## よく使うコマンド

| 目的 | コマンド |
| --- | --- |
| 開発環境を起動 | `docker compose up -d --build` |
| ログを追う | `docker compose logs -f frontend` |
| コンテナに入る | `docker compose exec frontend sh` |
| パッケージを追加 | `docker compose exec frontend bun add <pkg>` |
| 停止 | `docker compose down` |
| DB ごと初期化 | `docker compose down -v` |
| 本番相当ビルドを確認 | `docker compose --profile prod up -d --build frontend-prod db` |
| MySQL に接続 | `docker compose exec db mysql -u app -p app` |

本番相当の確認用サービス `frontend-prod` はポート 3001 で公開しています（開発用の 3000 と衝突しないため）。

## 設計上のポイント

### プロファイルによる dev / prod の切り替え

`frontend` は `dev`、`frontend-prod` は `prod` プロファイルに属します。
`.env` の `COMPOSE_PROFILES=dev` により、通常の `docker compose up` では開発用のみが起動します。
`db` はプロファイル指定なしのため、どちらの場合も常に起動します。

## API を追加するとき

1. `api/` にアプリのソースを置く
2. `docker/api/Dockerfile` を作成する
3. `compose.yaml` の `api` サービス（Go / Laravel 用のサンプルをコメントで用意済み）を有効化する
4. `frontend` の `depends_on` のコメントを外す

`api` は `depends_on.db.condition: service_healthy` を指定しているため、MySQL のヘルスチェック通過後に起動します。
マイグレーション実行時の「DB がまだ起動していない」問題を防げます。