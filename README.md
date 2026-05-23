# DevForge
IT企業における社員情報と案件情報を一元管理して、スキルの需要や習得率等を分析したりするのに役立てるスキルマネジメントプラットフォーム。

## 目次

1. [技術スタック](#技術スタック)
2. [ディレクトリ構成](#ディレクトリ構成)
3. [開発環境構築](#開発環境構築)
4. [コマンド](#コマンド)
5. [その他](#その他)

## 技術スタック

| 言語・フレームワーク  | バージョン |
| --------------------- | -------------- |
| React                 | 19.x           |
| Typescript            | 5.x (strict)   |
| Next.js               | 16.1.16        |
| shadcn/ui             |                |
| PHP                   | 8.4            |
| Laravel               | 13.x.          |
| MySQL                 | 8.4            |

## ディレクトリ構成

<pre>
.
├── api         # API実装
└── docker      # Dockerコンテナ定義
└── docs        # 設計原則やコーディング規約類のドキュメント
└── front       # UI実装
</pre>


## 開発環境構築

### 前提条件

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) がインストール済みで、起動していること
- Git がインストール済みであること

### 手順

#### 1. リポジトリのクローン

```bash
git clone <repository-url>
cd DevForge
```

#### 2. 環境変数ファイルの作成

```bash
cp .env.example .env
```

必要に応じて `.env` の値を編集してください。

#### 3. Docker イメージのビルドとコンテナ起動

```bash
docker compose up -d --build
```

#### 4. Laravel プロジェクトの初期化（初回のみ）

```bash
docker compose exec api php artisan key:generate
docker compose exec api php artisan migrate
```

#### 5. Laravel Boost 導入（初回のみ）

```bash
docker compose exec api composer require laravel/boost --dev
docker compose exec -it api php artisan boost:install
```

### アクセス先

| サービス | URL |
|---------|-----|
| フロントエンド（Next.js） | http://localhost:3000 |
| バックエンド（Laravel） | http://localhost |
| MySQL | localhost:3306 |

## コマンド

## その他

