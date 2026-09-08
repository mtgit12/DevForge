# Next.js 16 / React / TypeScript

> **バージョン**: Next.js 16 (App Router) / React 19 / TypeScript

---

## コマンド

| 目的 | コマンド |
| --- | --- |
| 依存関係のインストール | `bun install` |
| パッケージの追加 | `bun add <package>` |
| パッケージの追加（開発依存） | `bun add -d <package>` |
| 開発サーバー起動 | `bun run dev` |
| 本番ビルド | `bun run build` |
| 本番サーバー起動 | `bun run start` |
| 型チェック | `bun run typecheck` |
| Lint（Biome） | `bun run lint` |
| フォーマット（Biome, 自動修正） | `bun run format` |
| shadcn/ui コンポーネント追加 | `bunx shadcn@latest add <component>` |

---

## ディレクトリ構造

```
src/
├── app/                        # Next.js App Router
│   ├── (auth)/                 # 認証不要ページ群
│   │   └── login/
│   ├── admin/                  # 認証必要ページ群（管理側）
│   │   ├── employees/
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── user/                   # 認証必要ページ群（ユーザー側）
│   │   ├── employees/
│   │   ├── layout.tsx
│   │   └── page.tsx
│
├── features/                   # 機能別モジュール
│   ├── auth/
│   │   ├── components/         # コンポーネント
│   │   ├── hooks/              # カスタムフック
│   │   ├── stores/             # 状態管理
│   │   ├── types/              # 型定義
│   │   └── api/                # API通信関数
│   ├── admin/
│   │   ├── employees/
│   │   │   ├── components/
│   │   │   └── api/
│   │   ├── projects/
│   └── user/
│
├── components/                 # プロジェクト共通コンポーネント
│   ├── ui/                     # shadcn/ui コンポーネント置き場
│   └── common/                 # コンポーネント
│
├── lib/                        # ユーティリティ・設定
│   ├── api/                    # API通信基盤（axiosインスタンス等）
│   ├── utils/                  # 汎用ユーティリティ関数
│   └── constants/              # 定数定義
│
├── hooks/                      # グローバル共通カスタムフック
│
├── stores/                     # グローバル状態管理
│
└── types/                      # グローバル型定義
    ├── api/                    # APIレスポンス/リクエスト型（共通）
    ├── schema/                 # Zodスキーマ
    ├── model/                  # 基底型（モデル型）
    └── index.ts
```