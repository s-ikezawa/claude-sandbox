# YAGNI原則 (You Aren't Gonna Need It)

## クイックリファレンス(実装時のチェックリスト)

YAGNI原則適用時は以下をチェックしてください:

- [ ] この機能は今すぐ必要か?
- [ ] 「将来必要になるかも」という理由で実装していないか?
- [ ] 現在のテストや要件が本当に要求しているか?
- [ ] 仮定や推測に基づいて実装していないか?
- [ ] 使われていないコードや機能はないか?

### TDDとの関係
- **完全一致**: TDDは「テストが要求するもののみ実装」= YAGNI
- **Red段階**: 今必要なテストのみ書く
- **Green段階**: そのテストを通すために必要な最小限のコードのみ書く
- **Refactor段階**: 使われていないコードは削除

### 他の原則との関係
- **KISS原則**: 不要なものを削除することでシンプルになる（相乗効果）
- **DRY原則**: 使われないコードを共通化する意味はない（YAGNI > DRY）
- **SOLID原則**: 必要になってから適用する（YAGNIに従う）

---

## 概要

YAGNI原則は「You Aren't Gonna Need It（それは必要にならない）」の略で、**今必要でない機能は実装しない**という設計原則です。エクストリームプログラミング(XP)の実践の1つとして提唱されました。

### 核心的な考え方

> "Always implement things when you actually need them, never when you just foresee that you need them."
>
> 実際に必要になった時に実装しなさい。必要になると予見した時には実装してはいけない。

## YAGNI原則の重要性

### メリット

1. **開発時間の節約**: 使われない機能に時間を費やさない
2. **コードの削減**: コードベースが小さく保たれる
3. **保守コストの削減**: 不要なコードのメンテナンスが不要
4. **複雑性の削減**: システム全体がシンプルになる
5. **柔軟性の向上**: 実際の要件が明確になってから実装する方が適切な設計ができる
6. **テストの削減**: テストすべきコードが減る

### デメリット(誤解された場合)

1. **リファクタリングコスト**: 後から追加する際の変更コスト
2. **設計の見直し**: 将来の拡張を全く考慮しないと、大きな設計変更が必要になることも

**ただし**: TDDとペアで実践すれば、テストがあるためリファクタリングは安全に行えます。

## YAGNI違反のパターンと改善例

### パターン1: 将来の拡張を見越した過度な汎用化

#### ❌ 悪い例

```typescript
// 現在は文字列のフォーマットしか必要ないのに...
interface Formatter<T> {
  format(value: T): string;
}

class StringFormatter implements Formatter<string> {
  format(value: string): string {
    return value.trim();
  }
}

class NumberFormatter implements Formatter<number> {
  format(value: number): string {
    return value.toFixed(2);
  }
}

class DateFormatter implements Formatter<Date> {
  format(value: Date): string {
    return value.toISOString();
  }
}

// 現在は文字列しか使っていない!
const formatter: Formatter<string> = new StringFormatter();
```

#### ✅ 良い例

```typescript
// 今必要なものだけを実装
function formatString(value: string): string {
  return value.trim();
}

// 将来、数値のフォーマットが必要になったら、その時に追加すればよい
```

---

### パターン2: 使われない設定オプション

#### ❌ 悪い例

```typescript
interface UserServiceOptions {
  cacheEnabled?: boolean;        // 今は使われていない
  cacheTTL?: number;             // 今は使われていない
  retryCount?: number;           // 今は使われていない
  timeout?: number;              // 今は使われていない
  logLevel?: 'debug' | 'info';   // 今は使われていない
}

class UserService {
  constructor(private options: UserServiceOptions = {}) {}

  getUser(id: string): User {
    // 実際にはoptionsを使っていない
    return fetchUser(id);
  }
}
```

#### ✅ 良い例

```typescript
// 今必要なものだけ
class UserService {
  getUser(id: string): User {
    return fetchUser(id);
  }
}

// キャッシュが必要になったら、その時に追加
// class UserService {
//   constructor(private cache: Cache) {}
//   getUser(id: string): User {
//     return this.cache.get(id) || fetchUser(id);
//   }
// }
```

---

### パターン3: 将来のデータベース切り替えを見越した過度な抽象化

#### ❌ 悪い例

```typescript
// 現在はPostgreSQLしか使わないのに...
interface Database {
  connect(): Promise<void>;
  query(sql: string): Promise<any>;
  disconnect(): Promise<void>;
}

interface DatabaseFactory {
  createDatabase(type: 'postgres' | 'mysql' | 'mongodb'): Database;
}

class PostgresDatabase implements Database {
  async connect() { /* ... */ }
  async query(sql: string) { /* ... */ }
  async disconnect() { /* ... */ }
}

class MySQLDatabase implements Database {
  async connect() { /* ... */ }
  async query(sql: string) { /* ... */ }
  async disconnect() { /* ... */ }
}

// MongoDB は今後も使う予定がない!
class MongoDBDatabase implements Database {
  async connect() { /* ... */ }
  async query(sql: string) { /* ... */ }
  async disconnect() { /* ... */ }
}
```

#### ✅ 良い例

```typescript
// 今使うデータベースだけを実装
class Database {
  async connect() {
    // PostgreSQL接続
  }

  async query(sql: string) {
    // PostgreSQLクエリ実行
  }

  async disconnect() {
    // PostgreSQL切断
  }
}

// 将来、別のDBが必要になったら、その時にインターフェースを抽出すればよい
```

---

### パターン4: 使われていない引数やパラメータ

#### ❌ 悪い例

```typescript
// 将来使うかもしれないという理由で追加された引数
function createUser(
  name: string,
  email: string,
  age: number,
  address?: string,      // 今は使われていない
  phone?: string,        // 今は使われていない
  department?: string,   // 今は使われていない
  manager?: string       // 今は使われていない
): User {
  return {
    name,
    email,
    age
  };
  // address, phone, department, manager は使われていない!
}
```

#### ✅ 良い例

```typescript
// 今必要なものだけ
function createUser(name: string, email: string, age: number): User {
  return {
    name,
    email,
    age
  };
}

// 将来、addressが必要になったら、その時に追加
// function createUser(
//   name: string,
//   email: string,
//   age: number,
//   address: string
// ): User {
//   return { name, email, age, address };
// }
```

---

### パターン5: プラグインシステムの先行実装

#### ❌ 悪い例

```typescript
// プラグインを使う予定がないのに...
interface Plugin {
  name: string;
  execute(): void;
}

class PluginManager {
  private plugins: Plugin[] = [];

  register(plugin: Plugin): void {
    this.plugins.push(plugin);
  }

  executeAll(): void {
    this.plugins.forEach(p => p.execute());
  }
}

class App {
  private pluginManager = new PluginManager();

  run() {
    this.pluginManager.executeAll();
    // 実際にはプラグインを使っていない!
  }
}
```

#### ✅ 良い例

```typescript
// プラグインシステムなしで開始
class App {
  run() {
    // 必要な処理を直接実装
    this.doSomething();
  }

  private doSomething() {
    // 実装
  }
}

// プラグインが本当に必要になったら、その時にリファクタリング
```

---

## YAGNI適用のベストプラクティス

### 1. 現在の要件にフォーカス

```typescript
// ❌ 将来を予測
// 「将来、複数の通貨に対応するかもしれない」
class Price {
  constructor(
    private amount: number,
    private currency: string = 'USD'  // 今は全てUSD
  ) {}
}

// ✅ 今の要件
// 現在は全てUSDなので、通貨パラメータは不要
class Price {
  constructor(private amount: number) {}
}
```

### 2. テストが要求するものだけ実装(TDDとの連携)

```typescript
// Test
test('ユーザーを作成できる', () => {
  const user = createUser('太郎', 'taro@example.com');
  expect(user.name).toBe('太郎');
  expect(user.email).toBe('taro@example.com');
});

// ✅ テストが要求するものだけ実装
function createUser(name: string, email: string): User {
  return { name, email };
}

// ❌ テストが要求していないものまで実装
// function createUser(name: string, email: string): User {
//   return {
//     name,
//     email,
//     createdAt: new Date(),  // テストで要求されていない
//     updatedAt: new Date(),  // テストで要求されていない
//     id: generateId()        // テストで要求されていない
//   };
// }
```

### 3. 設定より規約(Convention over Configuration)

```typescript
// ❌ 過度な設定オプション
interface LoggerConfig {
  level?: 'debug' | 'info' | 'warn' | 'error';
  format?: 'json' | 'text';
  destination?: 'console' | 'file';
  filename?: string;
  maxFileSize?: number;
  rotationPolicy?: 'daily' | 'size';
}

class Logger {
  constructor(config: LoggerConfig = {}) {
    // 全ての設定を処理...
  }
}

// ✅ デフォルト値で開始、必要になったら設定を追加
class Logger {
  log(message: string) {
    console.log(message);  // シンプルに開始
  }
}
```

### 4. 段階的な進化

```typescript
// フェーズ1: 最小限の実装
class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
}

// フェーズ2: 必要になったら subtract を追加
class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }

  subtract(a: number, b: number): number {
    return a - b;
  }
}

// フェーズ3: 必要になったら multiply を追加
// ...という風に段階的に進化
```

### 5. デッドコードの削除

```typescript
// ❌ 使われていないコードを残す
class UserService {
  getUser(id: string): User {
    return fetchUser(id);
  }

  // この関数は誰も呼んでいない!
  getUserWithCache(id: string): User {
    // 将来使うかもしれないと思って残している
    return this.cache.get(id) || fetchUser(id);
  }
}

// ✅ 使われていないコードは削除
class UserService {
  getUser(id: string): User {
    return fetchUser(id);
  }
}

// 必要になったら git history から復元できる
```

## TDDとYAGNIの完璧な組み合わせ

### Red-Green-Refactorとの統合

#### Red段階
```typescript
// テスト: 今必要なテストのみ書く
test('ユーザー名を表示できる', () => {
  const user = { name: '太郎' };
  expect(displayUser(user)).toBe('太郎');
});

// ❌ 将来必要になるかもしれないテスト
// test('ユーザー名とメールアドレスを表示できる', () => {
//   // まだこの要件はない!
// });
```

#### Green段階
```typescript
// ✅ テストを通す最小限の実装
function displayUser(user: { name: string }): string {
  return user.name;
}

// ❌ テストが要求していない機能を追加
// function displayUser(user: {
//   name: string;
//   email?: string;
//   age?: number;
// }): string {
//   return `${user.name} (${user.email || 'no email'}) - ${user.age || 'unknown'}`;
// }
```

#### Refactor段階
```typescript
// 使われていない引数やコードがあれば削除
// 必要な機能だけを残す
```

### YAGNIとテストカバレッジ

```typescript
// ❌ 使われない機能のテスト
test('未来の機能: 通貨変換', () => {
  // 現在の要件にはない機能をテストしている
  expect(convertCurrency(100, 'USD', 'JPY')).toBe(11000);
});

// ✅ 現在必要な機能のみテスト
test('価格を表示できる', () => {
  expect(formatPrice(100)).toBe('$100');
});
```

## YAGNIを適用すべき/すべきでない状況

### YAGNI を適用すべき状況

1. **機能の実装時**
   - 「将来使うかも」という理由での実装は避ける

2. **設定オプションの追加時**
   - 今使わない設定は追加しない

3. **抽象化の検討時**
   - 1つのユースケースしかない段階での抽象化は避ける

### YAGNI を適用すべきでない状況

1. **セキュリティ関連**
   ```typescript
   // ✅ セキュリティは先に考慮すべき
   function hashPassword(password: string): string {
     return bcrypt.hashSync(password, 10);  // YAGNIでもセキュリティは重要
   }
   ```

2. **データベーススキーマ設計**
   ```sql
   -- ✅ 後から変更が困難な場合は事前設計が必要
   CREATE TABLE users (
     id UUID PRIMARY KEY,        -- 後から変更困難
     email VARCHAR(255) UNIQUE,  -- 後から変更困難
     created_at TIMESTAMP        -- 後から追加は可能だが事前に入れるのが自然
   );
   ```

3. **公開API設計**
   ```typescript
   // ✅ 後方互換性が必要なAPIは慎重に設計
   interface PublicAPI {
     getUser(id: string): User;  // 公開後は変更困難
   }
   ```

## YAGNIと他の原則のバランス

### YAGNI vs SOLID

```typescript
// 小規模プロジェクト: YAGNI優先
class UserService {
  getUser(id: string): User {
    return database.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}

// 大規模プロジェクト: SOLIDも考慮（ただし必要になってから）
interface UserRepository {
  findById(id: string): User;
}

class UserService {
  constructor(private repository: UserRepository) {}

  getUser(id: string): User {
    return this.repository.findById(id);
  }
}
```

### YAGNI vs DRY

```typescript
// 2箇所で似たコードがある時
function formatUserName(user: User): string {
  return `${user.firstName} ${user.lastName}`;
}

function formatAuthorName(author: Author): string {
  return `${author.firstName} ${author.lastName}`;
}

// YAGNI視点: まだ共通化する必要はない（2回程度の重複）
// DRY視点: 3回目が出現したら共通化を検討

// 3回目が出現した時点で共通化
function formatFullName(entity: { firstName: string; lastName: string }): string {
  return `${entity.firstName} ${entity.lastName}`;
}
```

## YAGNIのチェックリスト

実装前に以下を確認してください:

- [ ] この機能は現在のストーリー/チケットで要求されているか?
- [ ] この機能は現在のテストで必要とされているか?
- [ ] 「将来使うかも」という理由だけで実装しようとしていないか?
- [ ] この設定オプションは今すぐ使われるか?
- [ ] この抽象化は現在2つ以上のユースケースがあるか?
- [ ] このコードは実際に呼び出されているか?

実装後に以下を確認してください:

- [ ] 使われていないコードはないか?
- [ ] 使われていない引数はないか?
- [ ] 使われていない設定オプションはないか?
- [ ] 使われていない抽象化はないか?

## まとめ

### YAGNI原則の本質

1. **今必要なものだけ実装** - 将来の予測に基づいた実装は避ける
2. **TDDとの完璧な組み合わせ** - テストが要求するもののみ実装
3. **段階的な進化** - 必要になった時に追加する方が適切な設計ができる
4. **デッドコードの削除** - 使われていないコードは負債

### 心に留めておくべきこと

1. **予測は外れることが多い** - 「将来必要」と思ったものの多くは使われない
2. **変更は怖くない** - テストがあれば安全にリファクタリングできる
3. **シンプルから始める** - 複雑さは必要になってから追加する
4. **ただし例外はある** - セキュリティ、DB設計、公開APIは慎重に

### 関連する原則

- **KISS**: YAGNIに従えば自然とシンプルになる
- **TDD**: Red-Green-RefactorサイクルがYAGNIを自然に実現
- **DRY**: 使われていないコードを共通化する意味はない
- **SOLID**: 必要になってから適用する（YAGNI的アプローチ）
