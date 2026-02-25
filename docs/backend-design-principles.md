# アーキテクチャと設計原則

## 目次

1. [アーキテクチャの目的と基本方針](#1-アーキテクチャの目的と基本方針)
2. [ディレクトリ構造](#2-ディレクトリ構造)
3. [各レイヤーの設計原則とサンプルコード](#3-各レイヤーの設計原則とサンプルコード)

---

## 1. アーキテクチャの基本方針

### コアコンセプト

- **単一責任の原則（SRP）**: クラスは1つの役割のみ持つ
- **Fat Service の防止**: 値の検証・計算ロジックは Value Object に委譲し、Service の肥大化を防ぐ
- **関心の分離**: プレゼンテーション（Controller）/ アプリケーション（Service）/ データアクセス（Repository）を明確に分離する
- **DIP の適用**: Service は Repository のインターフェースに依存し、具体的な実装に依存しない

### レイヤー責務一覧

| レイヤー | 責務 | 禁止事項 |
|---|---|---|
| **Controller** | リクエスト受付・Policyによる認可・レスポンス返却 | ビジネスロジックの記述・直接のDB操作 |
| **FormRequest** | 入力値のバリデーションのみ | 認可チェック・ビジネスロジック |
| **Policy** | リソースへのアクセス権限チェック（認可） | バリデーション・ビジネスロジック |
| **Service** | ユースケースの進行役（オーケストレーター） | HTTPリクエスト/レスポンスの直接操作 |
| **Value Object** | 業務特有の「値」のルール・計算をカプセル化 | 状態を持つこと（Immutable） |
| **Repository** | DBアクセスの隠蔽・クエリ構築 | ビジネスロジックの記述 |
| **Model** | テーブル定義・リレーション・キャスト | 複雑なビジネスロジック |
| **API Resource** | レスポンスデータの整形 | DBアクセス |
| **Custom Exception** | ドメイン例外の定義・エラーコード管理 | - |

---

## 2. ディレクトリ構造

```
app/
├── Http/
│   ├── Controllers/
│   │   └── Api/
│   │       └── V1/                    # APIバージョン管理
│   │           └── XXXController.php
│   ├── Requests/                      # FormRequest（バリデーション・認可）
│   └── Resources/                     # API Resource（レスポンス整形）
│
├── Services/                          # アプリケーション層（ユースケース進行役）
│
├── ValueObjects/                      # ドメイン層（値オブジェクト）
│
├── Repositories/
│   ├── Contracts/                     # インターフェース（Serviceが依存する抽象）
│   │   └── XXXRepositoryInterface.php
│   └── Eloquent/                      # インターフェースの実装
│       └── XXXRepository.php
│
├── Models/                            # Eloquentモデル（DAOとして扱う）
│   └── XXX.php
│
├── Policies/                          # 認可ロジック（Laravel Policy）
│   └── XXXPolicy.php
│
├── Exceptions/
│   ├── Domain/                        # ドメイン例外
│   │   └── XXXException.php
│   └── Handler.php                    # 例外の一元処理
│
└── Providers/
    └── AppServiceProvider.php         # DIバインディング登録
```

---

## 3. 各レイヤールール

### ① Controller

**役割**: HTTPリクエストを受け取り、Service を呼び出し、API Resource を通じてレスポンスを返す。

**ルール**:
- ビジネスロジック（if文による状態判定等）は一切記述しない
- バリデーションは必ず `FormRequest` に委譲する
- **認可（Authorization）は必ず Policy を使い、`$this->authorize()` で呼び出す**
- DB操作は行わず、必ず Service 経由にする
- `try-catch` は不要（例外は `Handler.php` で一元処理）

---

### ② FormRequest

**役割**: 入力値のバリデーションのみをカプセル化する。認可チェックは行わない。

**ルール**:
- `authorize()` は常に `true` を返す（**認可は Controller の `$this->authorize()` + Policy で行う**）
- バリデーションルールは配列形式（文字列パイプ形式は避ける）

---

### ③ Service

**役割**: ユースケースの「進行役（オーケストレーター）」。状態の判定・Value Objectの生成・Repository への委譲・トランザクション管理を行う。

**ルール**:
- Repository のインターフェースに依存し、具体的実装に直接依存しない
- 「値そのもののルール」は Value Object に委譲する
- トランザクションが必要な処理は Service 内で `DB::transaction()` を使う

---

### ④ Value Object

**役割**: 業務特有の「値」のルールと振る舞いをカプセル化する。不正な値の生成を防ぎ、Service の肥大化（Fat Service）を防ぐ。

**ルール**:
- コンストラクタで不変条件を必ずチェックする
- Immutable（状態変更不可）にする
- 値を変更する場合は新しいインスタンスを返す

---

### ⑤ Repository

**役割**: DBアクセス（SQLの実行）を隠蔽し、Service がデータストアを意識しなくて済むようにする。

**ルール**:
- 必ず `Contracts/` にインターフェースを定義し、Service はインターフェースに依存させる
- 戻り値は Eloquent Model（またはそのコレクション）を許容する
- 生SQL は禁止。Eloquent / QueryBuilder を必ず使う

---

### ⑥ Model

**役割**: テーブルと1対1で対応するDAO。リレーション・キャスト・スコープのみ定義する。

**ルール**:
- `$fillable` でマスアサインメント保護（`$guarded = []` は使わない）
- 複雑なビジネスロジックは記述しない（純粋なデータの入れ物として扱う）
- N+1問題防止のため、呼び出し元で `with()` による Eager Loading を必ず行う


---

### ⑦ Custom Exception

**役割**: 例外設定。

**ルール**:
- ドメイン例外は `app/Exceptions/Domain/` 配下に機能別で定義する
- すべてのドメイン例外は `DomainException` を継承する
- HTTPステータスコードの対応は `Handler.php` で一元管理し、Controller には書かない

---
