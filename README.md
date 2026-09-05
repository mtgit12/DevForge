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

## 使用技術

### フロントエンド（`frontend/`）

| 分類 | 技術 | バージョン |
| --- | --- | --- |
| 言語 | TypeScript | ^5 |
| フレームワーク | Next.js（App Router / standalone 出力） | 16.3.4 |
| UI ライブラリ | React / React DOM | 19.2.8 |
| ランタイム | Bun | 1.4.1 |
| CSS | Tailwind CSS | ^4 |
| UI コンポーネント | shadcn/ui | ^4.21.0 |
| フォーム | React Hook Form | ^7.87.0 |
| バリデーション | Zod | ^4.5.4 |
| 状態管理 | Zustand | ^5.0.15 |
| Lint / Format | Biome | 2.4.2 |

バージョンは `frontend/package.json` を正としてください。

### バックエンド（`api/`）

現時点では未実装（将来追加）です。追加時は `docker/api/Dockerfile` と `compose.yaml` の `api` サービス（Go / Laravel のサンプルをコメントで用意済み）を有効化してください。詳細は後述の「API を追加するとき」を参照。

### インフラ

| 分類 | 技術 | バージョン |
| --- | --- | --- |
| コンテナ | Docker Compose | - |
| DB | MySQL | 8.4 |

## セットアップ（git clone からの環境構築）

前提: Docker / Docker Compose がインストールされていること。

```bash
# 1. リポジトリを取得
git clone git@github.com:mtgit12/DevForge.git
cd DevForge

# 2. 環境変数ファイルを用意
cp .env.example .env
# 必要に応じて .env の値（ポート番号や DB 認証情報など）を編集する

# 3. コンテナをビルドして起動（開発環境）
docker compose up -d --build

# 4. 起動確認
docker compose logs -f frontend
```

起動後、ブラウザで [http://localhost:3000](http://localhost:3000) にアクセスすると Next.js の開発サーバーが確認できます。

停止する場合は以下を実行してください。

```bash
docker compose down
```

DB のデータも含めて初期化したい場合は `docker compose down -v` を実行してください。

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