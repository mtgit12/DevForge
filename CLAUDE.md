# DevForge - Claude Code プロジェクトガイド

**バージョン**: 1.0  
**最終更新日**: 2025年12月30日  
**プロジェクト種別**: フルスタック スキルマネジメントプラットフォーム  
**開発体制**: 一人開発（フルスタック）

---

## 目次

1. [プロジェクト概要](#プロジェクト概要)
2. [技術スタック](#技術スタック)
3. [開発環境](#開発環境)
4. [プロジェクト構造](#プロジェクト構造)
5. [コーディング規約](#コーディング規約)
6. [コマンドリファレンス](#コマンドリファレンス)
7. [Git ワークフロー](#git-ワークフロー)
8. [テスト戦略](#テスト戦略)
9. [セキュリティガイドライン](#セキュリティガイドライン)
10. [エラーハンドリング・ログ戦略](#エラーハンドリングログ戦略)
11. [環境変数](#環境変数)
12. [トラブルシューティング](#トラブルシューティング)
13. [Claude への期待事項](#claude-への期待事項)

---

## プロジェクト概要

**DevForge** は、IT企業における社員のスキル・経歴情報と案件情報を一元管理し、効率的な人材配置とスキルマッチングを実現するスキルマネジメントプラットフォームです。開発者が自身のスキルを**鍛える（Forge）**ことを主眼に置き、過去の経歴管理から現在の案件アサインまでをシームレスに統合します。

### 主要機能

- **ロールベースアクセス制御（RBAC）**: エンジニア、人事・営業、管理者、システム運営
- **統合案件管理**: 社内案件と過去経歴を単一の `projects` テーブルで管理
- **スキルマッチング**: タグベースのスキルシステムとコンボボックス入力
- **ダッシュボード**: ロール別ビューとリアルタイムアサイン状況、需要急上昇スキル表示
- **PDF スキルシート出力**: SES業界標準フォーマット

### 現在のステータス

- **フェーズ**: 実装フェーズ（未着手）
- **要件定義書**: `/mnt/project/DevForge_要件定義書.md` で参照可能

---

## 技術スタック

### フロントエンド

- **フレームワーク**: Next.js 16 (App Router)
- **言語**: TypeScript
- **UI ライブラリ**: React 19
- **アーキテクチャ**: bulletproof-react パターン
- **テスト**: Vitest + React Testing Library
- **コード品質**: Biome
- **コンポーネント開発**: Storybook

### バックエンド

- **フレームワーク**: Laravel 12
- **言語**: PHP 8.4
- **アーキテクチャ**: MVC + Repository + Service + 軽量Clean Architecture
- **テスト**: PHPUnit
- **コード品質**: PHP-CS-Fixer

### インフラ

- **データベース**: MySQL 8.0+ (RDS)
- **認証**: Laravel Sanctum (SPA認証)
- **クラウド**: AWS (EC2, RDS, S3)
- **開発環境**: Docker
- **データベース管理**: phpMyAdmin

### パッケージ管理

- **フロントエンド**: npm
- **バックエンド**: Composer（Dockerコンテナ内で実行）

---

## 開発環境

### 必要な環境

- Docker & Docker Compose
- VSCode
- Node.js 20+（ローカルでnpmコマンド実行用）

### 初期セットアップ

```bash
# リポジトリをクローン
git clone <repository-url>
cd devforge

# 環境変数ファイルをコピー
cp .env.example .env
cd frontend && cp .env.local.example .env.local && cd ..

# Dockerコンテナを起動
docker compose up -d

# 依存関係をインストール
npm install
docker compose exec app composer install

# マイグレーションを実行
docker compose exec app php artisan migrate

# 初期データを投入（利用可能な場合）
docker compose exec app php artisan db:seed

# フロントエンド開発サーバーを起動
npm run dev

# アプリケーションにアクセス
# フロントエンド: http://localhost:3000
# バックエンドAPI: http://localhost:8000
# phpMyAdmin: http://localhost:8080
# Storybook: http://localhost:6006
```

### 開発ツール

- **VSCode**: メインIDE
  - 推奨拡張機能: ESLint, Prettier, PHP Intelephense, Tailwind CSS IntelliSense
- **phpMyAdmin**: データベース管理（http://localhost:8080）
- **Storybook**: コンポーネント開発・ドキュメント
- **Biome**: フロントエンド用の高速リンター・フォーマッター
- **PHP-CS-Fixer**: PHPコードスタイルフィクサー

---

## プロジェクト構造

### モノレポ構造

```
devforge/
├── frontend/                    # Next.jsアプリケーション
│   ├── src/
│   │   ├── app/                # App Routerページ
│   │   │   ├── (auth)/        # 認証グループ
│   │   │   ├── (dashboard)/   # ダッシュボードグループ
│   │   │   └── layout.tsx
│   │   ├── features/          # 機能ベースモジュール（bulletproof-react）
│   │   │   ├── auth/
│   │   │   │   ├── api/       # API呼び出し
│   │   │   │   ├── components/ # 機能コンポーネント
│   │   │   │   ├── hooks/     # カスタムフック
│   │   │   │   ├── types/     # TypeScript型定義
│   │   │   │   └── utils/     # ユーティリティ
│   │   │   ├── dashboard/
│   │   │   ├── profile/
│   │   │   ├── projects/
│   │   │   └── skills/
│   │   ├── components/        # 共有コンポーネント
│   │   │   ├── ui/           # 基本UIコンポーネント
│   │   │   ├── layouts/      # レイアウトコンポーネント
│   │   │   └── forms/        # フォームコンポーネント
│   │   ├── lib/              # 共有ライブラリ
│   │   │   ├── api/          # APIクライアント設定
│   │   │   ├── auth/         # 認証ユーティリティ
│   │   │   └── utils/        # 汎用ユーティリティ
│   │   ├── hooks/            # 共有フック
│   │   ├── types/            # 共有型定義
│   │   └── config/           # 設定ファイル
│   ├── public/
│   ├── .storybook/
│   └── package.json
│
├── backend/                    # Laravelアプリケーション
│   ├── app/
│   │   ├── Http/
│   │   │   ├── Controllers/  # リクエスト処理
│   │   │   ├── Middleware/
│   │   │   └── Requests/     # フォームバリデーション
│   │   ├── Services/         # ビジネスロジック
│   │   ├── Repositories/     # データアクセス層
│   │   ├── Models/           # Eloquentモデル
│   │   └── Exceptions/       # カスタム例外
│   ├── database/
│   │   ├── migrations/
│   │   ├── seeders/
│   │   └── factories/
│   ├── routes/
│   │   ├── api.php
│   │   └── web.php
│   ├── tests/
│   │   ├── Unit/
│   │   └── Feature/
│   └── composer.json
│
├── docker/
│   ├── nginx/
│   ├── php/
│   └── mysql/
├── docker-compose.yml
└── README.md
```

### 機能ベースアーキテクチャ（bulletproof-react）

各機能モジュールには以下を含める必要があります：

```
features/auth/
├── api/
│   ├── login.ts           # ログインAPI呼び出し
│   ├── logout.ts          # ログアウトAPI呼び出し
│   └── getCurrentUser.ts  # 現在のユーザー取得
├── components/
│   ├── LoginForm.tsx      # ログインフォームコンポーネント
│   └── UserMenu.tsx       # ユーザーメニューコンポーネント
├── hooks/
│   ├── useAuth.ts         # 認証フック
│   └── useUser.ts         # ユーザーデータフック
├── types/
│   └── index.ts           # 型定義
└── utils/
    └── validation.ts      # バリデーションユーティリティ
```

**重要な原則：**
- 各機能は自己完結している
- 共有コードは `src/components`、`src/lib`、`src/hooks` に配置
- コンポーネントは種類ではなく機能ごとに整理
- 関連するコードの検索と変更が容易

---

## コーディング規約

### React / Next.js

**公式ガイドライン：**
- https://ja.react.dev/learn/thinking-in-react
- https://ja.react.dev/reference/rules

**基本原則：**
```typescript
// ✅ 良い例：純粋なコンポーネント
export function UserCard({ user }: { user: User }) {
  return <div>{user.name}</div>
}

// ❌ 悪い例：レンダリング内での副作用
export function UserCard({ user }: { user: User }) {
  localStorage.setItem('lastUser', user.id) // 副作用！
  return <div>{user.name}</div>
}

// ✅ 良い例：トップレベルでのフック
function useUserData(id: string) {
  const [user, setUser] = useState(null)
  useEffect(() => {
    fetchUser(id).then(setUser)
  }, [id])
  return user
}

// ❌ 悪い例：条件付きフック
function useUserData(id: string) {
  if (!id) return null // フックは無条件に呼び出す必要がある！
  const [user, setUser] = useState(null)
  return user
}

// ✅ 良い例：リストに適切なkeyを使用
{users.map(user => (
  <UserCard key={user.id} user={user} />
))}

// ❌ 悪い例：インデックスをkeyとして使用
{users.map((user, index) => (
  <UserCard key={index} user={user} />
))}
```

**コンポーネント構造：**
```typescript
// インポート順序：
// 1. Reactのインポート
// 2. サードパーティのインポート
// 3. 内部インポート（絶対パス）
// 4. 相対インポート
// 5. 型のインポート

import { useState, useEffect } from 'react'
import { useRouter } from 'next/navigation'

import { Button } from '@/components/ui/button'
import { useAuth } from '@/features/auth/hooks/useAuth'

import type { User } from '@/types/user'

// Propsのインターフェース
interface UserProfileProps {
  userId: string
  onUpdate?: (user: User) => void
}

// 明示的な戻り値の型を持つコンポーネント
export function UserProfile({ userId, onUpdate }: UserProfileProps): JSX.Element {
  // フックを最初に
  const [user, setUser] = useState<User | null>(null)
  const { isAuthenticated } = useAuth()
  const router = useRouter()

  // エフェクト
  useEffect(() => {
    // 実装
  }, [userId])

  // イベントハンドラ
  const handleUpdate = () => {
    // 実装
  }

  // レンダリング
  return (
    <div>
      {/* JSX */}
    </div>
  )
}
```

### TypeScript

**Google TypeScript Style Guide：**
- https://google.github.io/styleguide/tsguide.html

**主要ルール：**
```typescript
// ✅ ほとんどの場合はInterfaceよりTypeを使用
type User = {
  id: string
  name: string
  email: string
}

// ✅ 明示的な戻り値の型
function getUser(id: string): Promise<User> {
  return api.get(`/users/${id}`)
}

// ❌ 'any'を避ける
function processData(data: any) { // 悪い！
  return data.value
}

// ✅ 適切な型を使用
function processData(data: UserData): string {
  return data.value
}

// ✅ Null安全性
function getUserName(user: User | null): string {
  return user?.name ?? 'Unknown'
}

// ✅ 型ガード
function isUser(obj: unknown): obj is User {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'name' in obj
  )
}
```

### PHP / Laravel

**PSR-2 標準：**
- https://www.infiniteloop.co.jp/docs/psr/psr-2-coding-style-guide.php

**Laravel ベストプラクティス：**
```php
<?php

namespace App\Services;

use App\Models\User;
use App\Repositories\UserRepository;
use Illuminate\Support\Facades\Hash;

// ✅ 型ヒントと戻り値の型（PHP 8.4）
class UserService
{
    public function __construct(
        private readonly UserRepository $userRepository
    ) {}

    public function createUser(array $data): User
    {
        // バリデーションはFormRequestで実施
        $user = $this->userRepository->create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => Hash::make($data['password']),
        ]);

        return $user;
    }

    public function getUserById(int $id): ?User
    {
        return $this->userRepository->findById($id);
    }
}

// ✅ Repositoryパターン
class UserRepository
{
    public function create(array $data): User
    {
        return User::create($data);
    }

    public function findById(int $id): ?User
    {
        return User::find($id);
    }

    public function getActiveUsers(): Collection
    {
        return User::where('is_active', true)->get();
    }
}

// ✅ バリデーション用のFormRequest
class CreateUserRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'unique:users'],
            'password' => ['required', 'string', 'min:8'],
        ];
    }
}

// ✅ Controller：薄く保ち、Serviceに委譲
class UserController extends Controller
{
    public function __construct(
        private readonly UserService $userService
    ) {}

    public function store(CreateUserRequest $request): JsonResponse
    {
        $user = $this->userService->createUser($request->validated());

        return response()->json($user, 201);
    }
}
```

**PHP 8.4 新機能（積極的に使用）：**
```php
<?php

// ✅ Property Hooks（PHP 8.4の新機能）
class User
{
    public string $name {
        set(string $value) {
            $this->name = trim($value);
        }
    }

    public string $email {
        get => strtolower($this->email);
        set(string $value) {
            if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
                throw new InvalidArgumentException('Invalid email');
            }
            $this->email = $value;
        }
    }
}

// ✅ 文字列キーでの配列展開
$defaults = ['a' => 1, 'b' => 2];
$custom = ['b' => 3, 'c' => 4];
$merged = [...$defaults, ...$custom]; // ['a' => 1, 'b' => 3, 'c' => 4]

// ✅ 最新のPHP機能を積極的に活用
```

### 命名規則

| 種類 | 規則 | 例 |
|------|-----------|---------|
| コンポーネント | PascalCase | `UserProfile.tsx` |
| 関数・変数 | camelCase | `getUserData()`, `isActive` |
| 定数 | UPPER_SNAKE_CASE | `MAX_UPLOAD_SIZE` |
| ルートファイル | kebab-case | `user-profile/page.tsx` |
| データベース | snake_case | `user_projects`, `created_at` |
| 型・インターフェース | PascalCase | `User`, `ProjectData` |
| フック | camelCaseで'use'から開始 | `useAuth()`, `useUserData()` |

---

## コマンドリファレンス

### Docker

```bash
# 全サービスを起動
docker compose up -d

# 全サービスを停止
docker compose down

# ログを表示
docker compose logs -f [service-name]

# コンテナを再ビルド
docker compose build
docker compose up -d

# コンテナ内でコマンドを実行
docker compose exec app [command]
```

### Laravel（バックエンド）

```bash
# マイグレーション
docker compose exec app php artisan migrate
docker compose exec app php artisan migrate:rollback
docker compose exec app php artisan migrate:fresh
docker compose exec app php artisan migrate:fresh --seed

# シーディング
docker compose exec app php artisan db:seed
docker compose exec app php artisan db:seed --class=UserSeeder

# ファイル生成コマンド
docker compose exec app php artisan make:model User -mfs
docker compose exec app php artisan make:controller UserController
docker compose exec app php artisan make:request CreateUserRequest
docker compose exec app php artisan make:service UserService
docker compose exec app php artisan make:repository UserRepository

# キャッシュ管理
docker compose exec app php artisan cache:clear
docker compose exec app php artisan config:clear
docker compose exec app php artisan route:clear
docker compose exec app php artisan view:clear

# 最適化（本番環境）
docker compose exec app php artisan config:cache
docker compose exec app php artisan route:cache
docker compose exec app php artisan view:cache

# テスト
docker compose exec app php artisan test
docker compose exec app php artisan test --filter UserServiceTest

# コード品質
docker compose exec app vendor/bin/php-cs-fixer fix
docker compose exec app vendor/bin/php-cs-fixer fix --dry-run --diff
```

### Composer（バックエンド）

```bash
# 依存関係をインストール
docker compose exec app composer install

# パッケージを追加
docker compose exec app composer require package-name

# パッケージを更新
docker compose exec app composer update

# オートローダーをダンプ
docker compose exec app composer dump-autoload
```

### Next.js（フロントエンド）

```bash
# 開発サーバー
npm run dev

# ビルド
npm run build

# 本番サーバー起動
npm start

# Biomeでリント
npm run lint
npm run lint:fix

# 型チェック
npm run type-check

# Vitestでテスト
npm run test
npm run test:watch
npm run test:coverage

# Storybook
npm run storybook
npm run build-storybook
```

### npm（フロントエンド）

```bash
# 依存関係をインストール
npm install

# パッケージを追加
npm install package-name
npm install -D package-name  # 開発依存関係

# パッケージを更新
npm update

# 古いパッケージを確認
npm outdated
```

### コード品質（推奨コマンド）

```bash
# フロントエンド：Biome
npm run lint              # コードスタイルをチェック
npm run lint:fix          # コードスタイルの問題を修正
npm run format            # コードをフォーマット
npm run format:check      # フォーマットをチェック

# バックエンド：PHP-CS-Fixer
docker compose exec app vendor/bin/php-cs-fixer fix                    # 全て修正
docker compose exec app vendor/bin/php-cs-fixer fix app/Services      # 特定のディレクトリを修正
docker compose exec app vendor/bin/php-cs-fixer fix --dry-run --diff  # 変更をプレビュー

# コミット前に全ての品質チェックを実行
npm run lint && npm run type-check && npm run test
docker compose exec app vendor/bin/php-cs-fixer fix --dry-run && docker compose exec app php artisan test
```

---

## Git ワークフロー

### ブランチ戦略

```
main（本番環境）
  └── develop（開発環境）
        ├── feature/add-user-profile
        ├── feature/implement-skill-matching
        └── fix/login-validation-error
```

**ブランチの種類：**
- `main`: 本番環境対応コード
- `develop`: 開発ブランチ（統合）
- `feature/*`: 新機能
- `fix/*`: バグ修正
- `hotfix/*`: 緊急の本番修正（mainからブランチ）

### ワークフロー

```bash
# 新機能の開始
git checkout develop
git pull origin develop
git checkout -b feature/add-user-profile

# 機能の作業...
git add .
git commit -m "feat: add user profile component"

# リモートにプッシュ
git push origin feature/add-user-profile

# レビュー後（必要な場合）、developにマージ
git checkout develop
git merge feature/add-user-profile
git push origin develop

# 機能ブランチを削除
git branch -d feature/add-user-profile
git push origin --delete feature/add-user-profile
```

### コミットメッセージ規約（Conventional Commits）

**フォーマット：**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**タイプ：**
- `feat`: 新機能
- `fix`: バグ修正
- `docs`: ドキュメントのみ
- `style`: コードスタイル変更（フォーマット、セミコロンなど）
- `refactor`: リファクタリング
- `perf`: パフォーマンス改善
- `test`: テストの追加や更新
- `chore`: メンテナンスタスク（依存関係、設定など）
- `ci`: CI/CD変更

**例：**
```bash
# シンプルな機能追加
git commit -m "feat: add user profile page"

# スコープ付き
git commit -m "feat(auth): implement login validation"

# バグ修正
git commit -m "fix: resolve null pointer in user service"

# 本文付き
git commit -m "feat: add skill matching algorithm

Implemented fuzzy matching for skill tags using Levenshtein distance.
Includes tests for edge cases."

# 破壊的変更
git commit -m "feat!: change API response format

BREAKING CHANGE: API now returns camelCase instead of snake_case"
```

### プリコミットチェックリスト

```bash
# 1. リンターを実行
npm run lint:fix
docker compose exec app vendor/bin/php-cs-fixer fix

# 2. テストを実行
npm run test
docker compose exec app php artisan test

# 3. 型チェック（フロントエンド）
npm run type-check

# 4. ステージングとコミット
git add .
git commit -m "feat: your message"
```

### 保存時のコードフォーマット（VSCode）

**settings.json：**
```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll": true,
    "source.organizeImports": true
  },
  "[typescript]": {
    "editor.defaultFormatter": "biomejs.biome"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "biomejs.biome"
  },
  "[php]": {
    "editor.defaultFormatter": "junstyle.php-cs-fixer"
  }
}
```

---

## テスト戦略

### 方針

**対象範囲：単体テストのみ**
- フロントエンド：コンポーネントロジック、フック、ユーティリティ
- バックエンド：Serviceレイヤー、リポジトリ、ビジネスロジック

**優先順位：**
1. **高優先度（必須）：**
   - 認証・認可ロジック
   - Serviceレイヤーのビジネスロジック
   - バリデーションロジック
   - データ変換・計算ロジック

2. **中優先度：**
   - Repositoryレイヤー
   - Utility関数
   - カスタムフック

3. **低優先度：**
   - シンプルなCRUD処理
   - 表示のみのコンポーネント

### フロントエンドテスト（Vitest + React Testing Library）

**テスト例：**
```typescript
// src/features/auth/hooks/useAuth.test.ts
import { renderHook, waitFor } from '@testing-library/react'
import { describe, it, expect, vi } from 'vitest'
import { useAuth } from './useAuth'

describe('useAuth', () => {
  it('認証済みの場合、ユーザーを返す', async () => {
    const { result } = renderHook(() => useAuth())

    await waitFor(() => {
      expect(result.current.isAuthenticated).toBe(true)
      expect(result.current.user).toBeDefined()
    })
  })

  it('ログインを正しく処理する', async () => {
    const { result } = renderHook(() => useAuth())

    await result.current.login('test@example.com', 'password')

    expect(result.current.isAuthenticated).toBe(true)
  })
})

// コンポーネントテスト
import { render, screen, fireEvent } from '@testing-library/react'
import { LoginForm } from './LoginForm'

describe('LoginForm', () => {
  it('無効な入力に対してバリデーションエラーを表示する', async () => {
    render(<LoginForm onSubmit={vi.fn()} />)

    const submitButton = screen.getByRole('button', { name: /login/i })
    fireEvent.click(submitButton)

    expect(await screen.findByText(/email is required/i)).toBeInTheDocument()
  })
})
```

### バックエンドテスト（PHPUnit）

**テスト例：**
```php
<?php

namespace Tests\Unit\Services;

use Tests\TestCase;
use App\Services\UserService;
use App\Repositories\UserRepository;
use Mockery;

class UserServiceTest extends TestCase
{
    private UserService $userService;
    private $userRepository;

    protected function setUp(): void
    {
        parent::setUp();
        $this->userRepository = Mockery::mock(UserRepository::class);
        $this->userService = new UserService($this->userRepository);
    }

    public function test_create_user_success(): void
    {
        // Arrange（準備）
        $data = [
            'name' => 'Test User',
            'email' => 'test@example.com',
            'password' => 'password123',
        ];

        $this->userRepository
            ->shouldReceive('create')
            ->once()
            ->andReturn((object)$data);

        // Act（実行）
        $user = $this->userService->createUser($data);

        // Assert（検証）
        $this->assertEquals('Test User', $user->name);
        $this->assertEquals('test@example.com', $user->email);
    }

    protected function tearDown(): void
    {
        Mockery::close();
        parent::tearDown();
    }
}

// Feature testの例
namespace Tests\Feature;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

class AuthTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_login(): void
    {
        $user = User::factory()->create([
            'email' => 'test@example.com',
            'password' => Hash::make('password123'),
        ]);

        $response = $this->postJson('/api/login', [
            'email' => 'test@example.com',
            'password' => 'password123',
        ]);

        $response->assertStatus(200);
        $this->assertAuthenticated();
    }
}
```

### テスト作成ガイドライン

**AAAパターン（Arrange-Act-Assert）：**
```typescript
it('合計を正しく計算する', () => {
  // Arrange（準備）：テストデータを設定
  const items = [
    { price: 100, quantity: 2 },
    { price: 50, quantity: 3 }
  ]

  // Act（実行）：関数を実行
  const total = calculateTotal(items)

  // Assert（検証）：結果を検証
  expect(total).toBe(350)
})
```

**ベストプラクティス：**
- テストは独立している必要がある（共有状態を持たない）
- わかりやすいテスト名を使用
- 外部依存をモック化
- エッジケースとエラー条件をテスト
- テストはシンプルで読みやすく保つ

---

## セキュリティガイドライン

### 認証・認可

```typescript
// フロントエンド：保護されたルート
import { redirect } from 'next/navigation'
import { getServerSession } from 'next-auth'

export default async function ProtectedPage() {
  const session = await getServerSession()

  if (!session) {
    redirect('/login')
  }

  // 保護されたコンテンツをレンダリング
}

// ロールベースアクセス
export function AdminPanel() {
  const { user } = useAuth()

  if (user.role !== 'admin') {
    return <div>アクセスが拒否されました</div>
  }

  return <div>管理者コンテンツ</div>
}
```

```php
// バックエンド：ミドルウェア
// routes/api.php
Route::middleware(['auth:sanctum'])->group(function () {
    Route::get('/user', [UserController::class, 'show']);
});

// 認可を使用したController
public function update(Request $request, User $user): JsonResponse
{
    $this->authorize('update', $user); // ポリシーチェック

    $user->update($request->validated());

    return response()->json($user);
}

// ポリシー
class UserPolicy
{
    public function update(User $authenticatedUser, User $user): bool
    {
        return $authenticatedUser->id === $user->id
            || $authenticatedUser->role === 'admin';
    }
}
```

### XSS対策

```typescript
// ✅ Reactはデフォルトで自動エスケープ
<div>{userInput}</div>

// ❌ 絶対に必要な場合以外はdangerouslySetInnerHTMLを使用しない
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ どうしても必要な場合は、まずサニタイズ
import DOMPurify from 'dompurify'
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />
```

```php
// ✅ Bladeは自動エスケープ
<div>{{ $userInput }}</div>

// ❌ エスケープなし（信頼できるコンテンツのみ）
<div>{!! $trustedHtml !!}</div>
```

### SQLインジェクション対策

```php
// ✅ EloquentまたはQuery Builderを使用（パラメータ化されたクエリ）
$users = User::where('email', $email)->get();
$users = DB::table('users')->where('email', $email)->get();

// ❌ ユーザー入力を含む生SQLは絶対に使用しない
$users = DB::select("SELECT * FROM users WHERE email = '$email'");

// ✅ 生SQLを使用する必要がある場合は、バインディングを使用
$users = DB::select('SELECT * FROM users WHERE email = ?', [$email]);
```

### ファイルアップロードのセキュリティ

```php
// ファイルアップロードのバリデーション
public function rules(): array
{
    return [
        'avatar' => [
            'required',
            'image',
            'mimes:jpeg,png,jpg',
            'max:2048', // 2MB
        ],
    ];
}

// 適切な権限でS3にアップロード
public function uploadAvatar(UploadedFile $file): string
{
    $filename = Str::uuid() . '.' . $file->extension();

    $path = Storage::disk('s3')->putFileAs(
        'avatars',
        $file,
        $filename,
        'public'
    );

    return Storage::disk('s3')->url($path);
}
```

### レート制限

```php
// routes/api.php
Route::middleware(['auth:sanctum', 'throttle:60,1'])->group(function () {
    Route::post('/projects', [ProjectController::class, 'store']);
});

// カスタムレート制限
Route::middleware(['auth:sanctum', 'throttle:api-heavy'])->group(function () {
    Route::post('/skill-sheet/generate', [SkillSheetController::class, 'generate']);
});

// config/app.php
'api-heavy' => [
    'limit' => 10,
    'decay' => 60, // 1分あたり10リクエスト
],
```

### 環境変数とシークレット管理

```bash
# ✅ .envに保存（決してコミットしない）
DB_PASSWORD=super_secret_password
AWS_SECRET_ACCESS_KEY=secret_key

# ❌ シークレットをハードコードしない
const apiKey = 'hardcoded-key-bad'  // 悪い例！

# ✅ 環境変数を使用
const apiKey = process.env.API_KEY
```

### CORS設定

```php
// config/cors.php
return [
    'paths' => ['api/*', 'sanctum/csrf-cookie'],
    'allowed_methods' => ['*'],
    'allowed_origins' => [
        env('FRONTEND_URL', 'http://localhost:3000')
    ],
    'allowed_headers' => ['*'],
    'exposed_headers' => [],
    'max_age' => 0,
    'supports_credentials' => true,
];
```

---

## エラーハンドリング・ログ戦略

### エラーハンドリング

#### フロントエンド

```typescript
// エラーバウンダリコンポーネント
'use client'

import { Component, type ReactNode } from 'react'

interface Props {
  children: ReactNode
  fallback?: ReactNode
}

interface State {
  hasError: boolean
  error?: Error
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props)
    this.state = { hasError: false }
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error }
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('バウンダリでキャッチされたエラー:', error, errorInfo)
    // エラーレポートサービスにログを送信（例：Sentry）
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div>
          <h2>問題が発生しました</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            再試行
          </button>
        </div>
      )
    }

    return this.props.children
  }
}

// APIエラーハンドリング
export async function fetchUser(id: string): Promise<User> {
  try {
    const response = await fetch(`/api/users/${id}`)

    if (!response.ok) {
      throw new ApiError(
        response.status,
        await response.json()
      )
    }

    return await response.json()
  } catch (error) {
    if (error instanceof ApiError) {
      // APIエラーを処理
      console.error('APIエラー:', error.message)
      throw error
    }

    // ネットワークエラーを処理
    console.error('ネットワークエラー:', error)
    throw new Error('ユーザーの取得に失敗しました')
  }
}

// カスタムエラークラス
export class ApiError extends Error {
  constructor(
    public status: number,
    public data: any
  ) {
    super(`APIエラー: ${status}`)
    this.name = 'ApiError'
  }
}
```

#### バックエンド

```php
<?php

namespace App\Exceptions;

use Exception;

// ビジネスロジックエラー用のカスタム例外
class BusinessLogicException extends Exception
{
    public function __construct(
        string $message,
        public readonly array $context = []
    ) {
        parent::__construct($message);
    }
}

// Serviceでの使用例
class UserService
{
    public function assignToProject(int $userId, int $projectId): void
    {
        $user = $this->userRepository->findById($userId);

        if (!$user) {
            throw new BusinessLogicException('ユーザーが見つかりません', [
                'user_id' => $userId
            ]);
        }

        if ($user->isAssignedToProject($projectId)) {
            throw new BusinessLogicException('ユーザーは既にこのプロジェクトにアサインされています', [
                'user_id' => $userId,
                'project_id' => $projectId
            ]);
        }

        // ユーザーをアサイン...
    }
}

// ハンドラー（app/Exceptions/Handler.php）
public function register(): void
{
    $this->renderable(function (BusinessLogicException $e, Request $request) {
        return response()->json([
            'message' => $e->getMessage(),
            'code' => 'BUSINESS_LOGIC_ERROR',
            'context' => $e->context
        ], 422);
    });
}
```

### ログ戦略

**ログレベル：**
- **DEBUG**: 開発時のデバッグ情報
- **INFO**: 通常の動作（ログイン、データ作成など）
- **WARNING**: 警告条件（非推奨機能の使用など）
- **ERROR**: エラー条件（APIエラー、DBエラーなど）
- **CRITICAL**: 重大な条件（システム障害レベル）

**フロントエンドログ：**
```typescript
// lib/logger.ts
export const logger = {
  debug: (message: string, data?: any) => {
    if (process.env.NODE_ENV === 'development') {
      console.debug(`[DEBUG] ${message}`, data)
    }
  },

  info: (message: string, data?: any) => {
    console.info(`[INFO] ${message}`, data)
  },

  warn: (message: string, data?: any) => {
    console.warn(`[WARN] ${message}`, data)
  },

  error: (message: string, error?: Error, data?: any) => {
    console.error(`[ERROR] ${message}`, error, data)
    // エラーレポートサービスに送信
  }
}

// 使用例
logger.info('ユーザーがログインしました', { userId: user.id })
logger.error('データ取得に失敗しました', error, { endpoint: '/api/users' })
```

**バックエンドログ：**
```php
<?php

use Illuminate\Support\Facades\Log;

// Controller
public function store(CreateUserRequest $request): JsonResponse
{
    try {
        Log::info('新しいユーザーを作成中', [
            'email' => $request->input('email')
        ]);

        $user = $this->userService->createUser($request->validated());

        Log::info('ユーザーが正常に作成されました', [
            'user_id' => $user->id
        ]);

        return response()->json($user, 201);
    } catch (Exception $e) {
        Log::error('ユーザー作成に失敗しました', [
            'error' => $e->getMessage(),
            'trace' => $e->getTraceAsString()
        ]);

        throw $e;
    }
}

// config/logging.php
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['single', 'daily'],
    ],
    'daily' => [
        'driver' => 'daily',
        'path' => storage_path('logs/laravel.log'),
        'level' => env('LOG_LEVEL', 'debug'),
        'days' => 14,
    ],
],
```

**ログに記録してはいけない情報：**
- パスワード
- トークン（アクセス、リフレッシュ、CSRF）
- クレジットカード情報
- 個人識別番号
- その他の機密データ

---

## 環境変数

### フロントエンド（.env.local）

```bash
# API設定
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_API_TIMEOUT=30000

# 認証
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-secret-key-here

# AWS S3（フロントエンドからアクセスする場合）
NEXT_PUBLIC_AWS_REGION=ap-northeast-1
NEXT_PUBLIC_AWS_BUCKET=devforge-uploads

# 機能フラグ
NEXT_PUBLIC_ENABLE_STORYBOOK=true
NEXT_PUBLIC_ENABLE_DEBUG_MODE=false

# アナリティクス（必要な場合）
NEXT_PUBLIC_GA_TRACKING_ID=UA-XXXXXXXXX-X

# 環境
NODE_ENV=development
```

### バックエンド（.env）

```bash
# アプリケーション
APP_NAME=DevForge
APP_ENV=local
APP_KEY=base64:your-app-key-here
APP_DEBUG=true
APP_URL=http://localhost:8000
FRONTEND_URL=http://localhost:3000

# データベース
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=devforge
DB_USERNAME=devforge_user
DB_PASSWORD=secure_password

# Laravel Sanctum
SANCTUM_STATEFUL_DOMAINS=localhost:3000,127.0.0.1:3000
SESSION_DOMAIN=localhost

# キャッシュ＆キュー
CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379

# メール（必要な場合）
MAIL_MAILER=smtp
MAIL_HOST=mailhog
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=noreply@devforge.local
MAIL_FROM_NAME="${APP_NAME}"

# AWS S3
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_DEFAULT_REGION=ap-northeast-1
AWS_BUCKET=devforge-uploads
AWS_USE_PATH_STYLE_ENDPOINT=false

# ログ
LOG_CHANNEL=stack
LOG_LEVEL=debug
LOG_DEPRECATIONS_CHANNEL=null

# セッション
SESSION_DRIVER=cookie
SESSION_LIFETIME=120
SESSION_ENCRYPT=false
SESSION_PATH=/
SESSION_SAME_SITE=lax

# セキュリティ
BCRYPT_ROUNDS=12
```

### Docker Compose環境

```yaml
# docker-compose.yml
services:
  app:
    environment:
      - DB_HOST=mysql
      - DB_DATABASE=devforge
      - DB_USERNAME=devforge_user
      - DB_PASSWORD=secure_password

  mysql:
    environment:
      - MYSQL_DATABASE=devforge
      - MYSQL_USER=devforge_user
      - MYSQL_PASSWORD=secure_password
      - MYSQL_ROOT_PASSWORD=root_password

  redis:
    environment:
      - REDIS_PASSWORD=
```

---

## トラブルシューティング

### よくある問題と解決方法

#### Docker関連の問題

**問題：コンテナが起動しない**
```bash
# ログを確認
docker compose logs app

# コンテナを再ビルド
docker compose down
docker compose build --no-cache
docker compose up -d
```

**問題：ポートが既に使用されている**
```bash
# ポートを使用しているプロセスを見つける
lsof -i :3000
lsof -i :8000

# プロセスを終了するか、docker-compose.ymlでポートを変更
```

#### データベース関連の問題

**問題：マイグレーションが失敗する**
```bash
# データベース接続を確認
docker compose exec app php artisan tinker
>>> DB::connection()->getPdo();

# データベースをリセット
docker compose exec app php artisan migrate:fresh

# MySQLログを確認
docker compose logs mysql
```

**問題：権限エラー**
```bash
# ストレージの権限を修正
docker compose exec app chmod -R 775 storage bootstrap/cache
docker compose exec app chown -R www-data:www-data storage bootstrap/cache
```

#### Laravel関連の問題

**問題：419 CSRFトークン不一致**
```php
// .envのSANCTUM_STATEFUL_DOMAINSを確認
SANCTUM_STATEFUL_DOMAINS=localhost:3000,127.0.0.1:3000

// 設定キャッシュをクリア
docker compose exec app php artisan config:clear

// フロントエンドがcredentialsを送信していることを確認
fetch('/api/endpoint', {
  credentials: 'include'
})
```

**問題：クラスが見つからない**
```bash
# オートロードをダンプ
docker compose exec app composer dump-autoload

// 全てのキャッシュをクリア
docker compose exec app php artisan optimize:clear
```

#### Next.js関連の問題

**問題：モジュールが見つからない**
```bash
# .nextディレクトリをクリア
rm -rf .next

# 依存関係を再インストール
rm -rf node_modules package-lock.json
npm install

# 開発サーバーを再起動
npm run dev
```

**問題：TypeScriptエラー**
```bash
# tsconfig.jsonを確認
# VSCodeでTypeScriptサーバーを再起動：Cmd+Shift+P -> "TypeScript: Restart TS Server"

# 型チェックを実行
npm run type-check
```

#### テスト関連の問題

**問題：データベースエラーでテストが失敗する**
```bash
# テストデータベースの設定を確認
# phpunit.xmlまたは.env.testing

# テストデータベースでマイグレーションを実行
docker compose exec app php artisan migrate --env=testing
```

#### パフォーマンス関連の問題

**問題：レスポンスが遅い**
```bash
# N+1クエリを確認（Laravel）
composer require barryvdh/laravel-debugbar --dev

# クエリログを有効化
DB::enableQueryLog();
// ... コード ...
dd(DB::getQueryLog());

// フロントエンドのバンドルを最適化
npm run build
npm run analyze  # 設定されている場合
```

### デバッグのヒント

**Laravelデバッグ：**
```bash
# Tinkerで簡単なテスト
docker compose exec app php artisan tinker
>>> User::first();

# デバッグモードを有効化
APP_DEBUG=true

# dd()とdump()を使用
dd($variable);  // Die and dump
dump($variable);  // dumpのみ
```

**Next.jsデバッグ：**
```typescript
// console.logを戦略的に使用
console.log('Debug:', { variable })

// React DevToolsを使用
// ブラウザ拡張機能をインストール

// API呼び出しのネットワークタブを確認
// ブラウザDevToolsのNetworkタブを使用
```

### AskUserQuestionToolを使用するタイミング

Claudeは以下の場合に`AskUserQuestionTool`を使用すべきです：
1. 要件が曖昧または不完全な場合
2. 複数の実装アプローチが可能な場合
3. ビジネスロジックの明確化が必要な場合
4. ユーザーの好み（例：UIデザインの選択）が必要な場合
5. セキュリティへの影響についてユーザーの決定が必要な場合
6. 技術的なトレードオフについて議論が必要な場合

**例：**
- 「このバリデーションはクライアント側とサーバー側のどちらで行うべきですか？」
- 「ユーザーが満員のプロジェクトに自分自身をアサインしようとした場合、どうすべきですか？」
- 「ダッシュボードのリアルタイム更新とポーリング、どちらが良いですか？」

---

## Claude への期待事項

### 基本原則

1. **シンプルさ優先**：巧妙なソリューションよりシンプルで読みやすいコードを書く
2. **標準準拠**：上記で定義されたコーディング標準に常に従う
3. **セキュリティ意識**：すべての機能でセキュリティへの影響を考慮
4. **型安全性**：TypeScriptとPHPの型システムを最大限活用
5. **テスト可能性**：テストしやすいコードを書く
6. **最新機能**：PHP 8.4とNext.js 16の新機能を積極的に使用

### サポート範囲

Claudeは以下の全般的なサポートを提供すべきです：

1. **コード実装**
   - クリーンでメンテナンス可能なコード
   - 標準準拠の実装
   - 適切なコメントとドキュメント

2. **設計ドキュメント**
   - API仕様書
   - データベース設計書
   - コンポーネント設計書

3. **コードレビュー**
   - セキュリティ観点のレビュー
   - パフォーマンス観点のレビュー
   - 可読性・保守性の改善提案

4. **バグ修正**
   - 根本原因の特定
   - 修正方法の提案
   - 再発防止策の提示

5. **テストコード**
   - 単体テストの実装
   - テストケースの提案
   - モックの適切な実装

6. **API設計**
   - RESTful APIの設計
   - エンドポイント命名規則
   - リクエスト・レスポンス形式

7. **データベース設計**
   - テーブル設計のレビュー
   - インデックス戦略
   - パフォーマンス最適化

### コード生成ガイドライン

コードを生成する際、Claudeは：

✅ **すべきこと：**
- すべてのコーディング標準を厳守
- 適切なエラーハンドリングを含める
- 複雑なロジックにコメントを追加
- 適切なTypeScript/PHPの型を使用
- セキュリティへの影響を考慮
- テスト可能なコードを書く
- 最新の言語機能を使用（PHP 8.4、ES2024）
- コンテキストと説明を提供
- 可能な場合は代替案を提案

❌ **すべきでないこと：**
- TypeScriptで`any`型を使用
- 機密値をハードコード
- エラーハンドリングをスキップ
- 不明瞭な変数名を書く
- 密結合なコードを作成
- エッジケースを無視
- 非推奨の機能を使用

### コミュニケーションスタイル

アドバイスや説明を提供する際：

1. **初心者向け**：専門用語を説明
2. **具体例を提供**：抽象的な説明だけでなく、具体的なコード例を提示
3. **理由を説明**：「なぜそうするのか」を明確に
4. **代替案を提示**：可能であれば複数の選択肢を提示
5. **ベストプラクティスに従う**：業界標準のプラクティスを優先
6. **質問する**：要件が不明確な場合は`AskUserQuestionTool`を使用
7. **簡潔に**：不必要な冗長さを避ける
8. **実用的に**：現実世界での適用可能性に焦点を当てる

### レスポンス形式

実装リクエストに対しては、以下を提供：

1. **概要**：アプローチの簡単な説明
2. **コード**：クリーンでコメント付きの実装
3. **テスト**：重要なロジックの単体テスト
4. **使用例**：必要に応じて使用例
5. **注意点**：重要な考慮事項や代替案

例：
```
## 概要
Laravel SanctumをSPA認証に使用してユーザー認証サービスを実装します。

## 実装
[コードをここに]

## テスト
[テストコードをここに]

## 使用例
[使用例をここに]

## 注意点
- .envのSANCTUM_STATEFUL_DOMAINSを設定することを忘れずに
- ログイン試行のレート制限を追加することを検討
```

---

## 開発フェーズ

### Phase 1：認証基盤（最優先）

**タスク：**
1. Laravel Sanctumのセットアップ
2. ユーザーモデル＆マイグレーション
3. ロール＆権限テーブル（RBAC）
4. 認証APIエンドポイント（ログイン、ログアウト、ユーザー情報）
5. フロントエンド認証フロー
6. 保護されたルート
7. ロールベースUIコンポーネント

**完了基準：**
- ユーザーがログイン/ログアウトできる
- ロールベースアクセスが機能する
- トークン管理が機能する
- 認証フローのテストが通る

### Phase 2：ダッシュボード

**タスク：**
1. ロール別ダッシュボードコンポーネント
2. エンジニアビュー：アサイン状況
3. エンジニアビュー：需要急上昇スキルランキング
4. 営業・管理者ビュー：稼働率、アラート
5. ダッシュボードAPIエンドポイント
6. リアルタイム更新（該当する場合）

**完了基準：**
- 各ロールが適切なダッシュボードを表示
- データが正しく表示される
- パフォーマンスが許容範囲内（<2秒ロード）

### Phase 3：プロフィール・経歴管理

**タスク：**
1. プロフィールCRUD（基本情報）
2. S3へのアバターアップロード
3. 職務経歴（My Histories）CRUD
4. スキルタグ管理（コンボボックス）
5. PDFスキルシート生成

**完了基準：**
- ユーザーがプロフィールを管理できる
- 画像が正常にアップロードされる
- 職務経歴が適切に記録される
- PDFが正しいフォーマットで生成される

### Phase 4：案件管理

**タスク：**
1. 案件マスタCRUD
2. 検索＆フィルタリング機能
3. アサイン管理（即時確定）
4. スキルマッチングアルゴリズム

**完了基準：**
- 案件を作成/管理できる
- 検索が効率的に機能する
- アサインが即座に確定する
- マッチングアルゴリズムが良好に機能する

---

## クイックリファレンス

### 重要なURL（開発環境）

- フロントエンド：http://localhost:3000
- バックエンドAPI：http://localhost:8000
- phpMyAdmin：http://localhost:8080
- Storybook：http://localhost:6006

### 主要ファイル

```
# 設定
frontend/.env.local          # フロントエンド環境変数
backend/.env                 # バックエンド環境変数
docker-compose.yml           # Docker設定

# ドキュメント
/mnt/project/DevForge_要件定義書.md  # 要件定義書
claude.md                    # このファイル

# 重要なLaravelファイル
backend/routes/api.php       # APIルート
backend/config/cors.php      # CORS設定
backend/config/sanctum.php   # Sanctum設定

# 重要なNext.jsファイル
frontend/src/app/layout.tsx  # ルートレイアウト
frontend/src/lib/api.ts      # APIクライアント
```

### 便利なリンク

- [要件定義書](/mnt/project/DevForge_要件定義書.md)
- [bulletproof-react](https://github.com/alan2207/bulletproof-react)
- [React 公式ドキュメント](https://ja.react.dev)
- [Next.js ドキュメント](https://nextjs.org/docs)
- [Laravel ドキュメント](https://laravel.com/docs/11.x)
- [PHP 8.4 機能](https://www.php.net/releases/8.4/en.php)
- [Conventional Commits](https://www.conventionalcommits.org)

---

## 注意事項

- これは**一人開発**プロジェクトです - シンプルさとメンテナンス性を優先
- 疑問がある場合は**質問する** - `AskUserQuestionTool`を使用
- **学習**に焦点を当てる - PHP 8.4とNext.js 16の新機能を積極的に使用
- コードは**シンプルで読みやすく** - 巧妙だが複雑なソリューションよりも優先
- **セキュリティ優先** - 常にセキュリティへの影響を考慮
- **重要なロジックをテスト** - すべてをテストする必要はないが、重要な部分はテストする
- **進めながらドキュメント化** - 良いコメントは未来の自分を助ける

---

**最終更新**：2025年12月30日  
**管理者**：DevForge 開発チーム  
**質問がありますか？**：明確化のために `AskUserQuestionTool` を使用してください