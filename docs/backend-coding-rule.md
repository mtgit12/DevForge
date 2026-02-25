# コーディング規約

## 命名規則

| 対象 | 規則 | 例 |
|---|---|---|
| クラス名 | PascalCase | `EmployeeService` |
| メソッド名 | camelCase | `assignSkill()` |
| 変数名 | camelCase | `$employeeList` |
| 定数 | UPPER_SNAKE_CASE | `MAX_SKILL_LEVEL` |
| インターフェース | PascalCase + `Interface` サフィックス | `EmployeeRepositoryInterface` |
| Value Object | PascalCase（業務用語に合わせる） | `SkillLevel`, `YearMonth` |
| Policy | PascalCase + `Policy` サフィックス | `EmployeePolicy`, `SkillPolicy` |
| テーブル名 | snake_case・複数形 | `employee_skills` |
| カラム名 | snake_case | `joined_at`, `is_active` |
| ルート名 | ドット区切り | `api.v1.employees.index` |
| マイグレーション | snake_case | `create_employees_table` |

---

## コーディング規約

### 基本

- 1クラス1ファイル
- `<?php` のみ（`?>` は書かない）
- [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md)コーディング標準に準拠

### 型宣言（PHP 8.4）

```php
// ✅ 必ず戻り値型・引数型を明示する
public function findById(int $id): Employee { }
public function findByEmail(string $email): ?Employee { }

// ✅ コンストラクタプロモーションを活用
public function __construct(
    private readonly EmployeeRepositoryInterface $repository,
    private readonly SkillService $skillService,
) {}

// ✅ readonly プロパティを積極活用
final class SkillLevel
{
    private readonly int $level;
}

// ❌ mixed・any は原則禁止（Larastanの型チェックを通す）
public function process(mixed $data): mixed { }
```

### その他

- 配列は短縮構文 `[]` を使用（`array()` 禁止）
- 文字列はシングルクォート優先（変数展開が必要な場合のみダブルクォート）
- 比較は `===` を使用（`==` は原則禁止）
- `use` でクラスをインポート（FQCN をコード内に直接書かない）
- `@return array<string, mixed>` 等のPHPDocによる型注釈を積極的に付ける（Larastan対応）

---