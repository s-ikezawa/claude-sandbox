# DRY原則(Don't Repeat Yourself)

## クイックリファレンス(実装時のチェックリスト)

DRY原則適用時は以下をチェックしてください:

- [ ] 同じコードが3回以上出現していないか?
- [ ] マジックナンバー/文字列を定数化しているか?
- [ ] 設定値を一箇所にまとめているか?
- [ ] バリデーションロジックは共通化されているか?
- [ ] エラーハンドリングは統一されているか?
- [ ] 共通化が過度になっていないか?
- [ ] 偶然の重複を無理に共通化していないか?

### TDDとの関係
- **Green段階では重複があっても問題ない**: テストを通すことに集中
- **Refactor段階でDRY原則を適用**: すべてのテストがGreenになってから
- **3回ルール**: 2回程度の重複なら様子見、3回目で共通化を検討
- テストがあるので、後からでも安全に共通化できる

### 他の原則との関係
- **KISS原則**: 過度な共通化は複雑さを増す（バランスが重要、[判断ガイド](dry-vs-kiss-decision-guide.md)参照）
- **YAGNI原則**: 使われないコードを共通化する意味はない（YAGNI > DRY）

---

## 概要

DRY原則は「同じことを繰り返すな」という設計原則です。Andy HuntとDave Thomasが著書『The Pragmatic Programmer』で提唱しました。

### 核心的な考え方

> "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system."
>
> システム内のすべての知識は、単一で明確な、信頼できる表現を持たなければならない。

## DRY原則の重要性

### メリット

1. **保守性の向上**: 変更が必要な場合、1箇所を修正するだけで済む
2. **バグの削減**: 重複コードがないため、修正漏れによるバグが減る
3. **可読性の向上**: コードが簡潔になり理解しやすくなる
4. **テストの容易性**: テスト対象が集約されるため、テストが書きやすい
5. **一貫性の保証**: ロジックが1箇所に集約されるため、挙動が一貫する

### デメリット(過度な適用の場合)

1. **過度な抽象化**: 無理に共通化すると、かえって複雑になる
2. **依存関係の増加**: 共通化により、モジュール間の結合度が高まる可能性
3. **パフォーマンスの低下**: 抽象化レイヤーが増えることでオーバーヘッドが発生する場合がある

## DRY違反の代表的なパターン

### パターン1: 同じコードの重複

```typescript
// ❌ 悪い例
function displayAdminUser(userId: string) {
  const user = database.query(`SELECT * FROM users WHERE id = ${userId}`);
  if (!user) return null;
  console.log(`Name: ${user.name}, Email: ${user.email}`);
  return user;
}

function displayNormalUser(userId: string) {
  // 同じコードの繰り返し
  const user = database.query(`SELECT * FROM users WHERE id = ${userId}`);
  if (!user) return null;
  console.log(`Name: ${user.name}, Email: ${user.email}`);
  return user;
}

// ✅ 良い例
function getUser(userId: string): User | null {
  const user = database.query(`SELECT * FROM users WHERE id = ${userId}`);
  if (!user) return null;
  return user;
}

function displayUser(user: User): void {
  console.log(`Name: ${user.name}, Email: ${user.email}`);
}
```

### パターン2: マジックナンバー/文字列の重複

```typescript
// ❌ 悪い例
function calculateDiscount(price: number, customerType: string): number {
  if (customerType === 'premium') return price * 0.8;  // 20%割引
  if (customerType === 'regular') return price * 0.9;  // 10%割引
  return price;
}

// ✅ 良い例
const DISCOUNT_RATES = {
  premium: 0.2,
  regular: 0.1,
  guest: 0
} as const;

function calculateDiscount(price: number, customerType: string): number {
  const rate = DISCOUNT_RATES[customerType] ?? 0;
  return price * (1 - rate);
}
```

### パターン3: バリデーションロジックの重複

```typescript
// ❌ 悪い例
function validateUserEmail(email: string): boolean {
  if (!email) return false;
  if (!email.includes('@')) return false;
  if (!email.includes('.')) return false;
  return true;
}

function validateAdminEmail(email: string): boolean {
  if (!email) return false;
  if (!email.includes('@')) return false;
  if (!email.includes('.')) return false;
  if (!email.endsWith('@company.com')) return false;
  return true;
}

// ✅ 良い例
function isValidEmailFormat(email: string): boolean {
  if (!email) return false;
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

function isEmailFromDomain(email: string, domain: string): boolean {
  return email.endsWith(`@${domain}`);
}

function validateUserEmail(email: string): boolean {
  return isValidEmailFormat(email);
}

function validateAdminEmail(email: string): boolean {
  return isValidEmailFormat(email) && isEmailFromDomain(email, 'company.com');
}
```

## 詳細なコード例

さらに多くのパターンと詳細な例については、以下を参照してください:
- [DRY原則 - 詳細なコード例とパターン](dont-repeat-yourself-examples.md)

## 3回ルール

同じコードが3回出現したら、共通化を検討する。2回程度なら様子を見る。

**TDD実装時の注意**:
- Green段階で重複が発生するのは自然なこと
- まだ2回の重複なら、次のテストケースの実装を優先
- 3回目の重複が発生したら、Refactor段階で共通化を検討
- テストがあるため、後から安全に共通化可能

## DRYを適用すべきでない場合

### 1. 偶然の重複

```typescript
// これらは偶然似ているだけで、本質的には異なるロジック
function calculateOrderTotal(items: OrderItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

function calculateInventoryValue(products: Product[]): number {
  return items.reduce((sum, product) => sum + product.cost * product.stock, 0);
}

// 無理に共通化すると、かえって複雑になる
```

### 2. ドメインが異なる場合

ユーザー認証とAPI認証は似ているが、ドメインが異なるため、無理に共通化すべきではありません。

### 3. 将来的に異なる変更が予想される場合

現時点では同じロジックでも、将来的に変更される可能性が高い場合は、あえて分離しておく方が良い場合があります。

## まとめ

### DRY原則のチェックリスト

- [ ] 同じコードが3回以上出現していないか?
- [ ] マジックナンバー/文字列を定数化しているか?
- [ ] 設定値を一箇所にまとめているか?
- [ ] バリデーションロジックは共通化されているか?
- [ ] エラーハンドリングは統一されているか?
- [ ] 共通化が過度になっていないか?
- [ ] 抽象化のレベルは適切か?
- [ ] 偶然の重複を無理に共通化していないか?

### 心に留めておくべきこと

1. **DRYは手段であり、目的ではない** - コードの保守性向上が真の目的
2. **過度なDRYは避ける** - WET(Write Everything Twice)の方が良い場合もある
3. **ビジネスロジックの重複に注目** - 些細な重複よりも重要なロジックの重複を優先
4. **チーム全体で共有** - 共通化したコードはチーム全員が理解できる必要がある
5. **継続的なリファクタリング** - 一度で完璧にする必要はなく、段階的に改善する

### 関連する設計原則

- **KISS(Keep It Simple, Stupid)**: シンプルさを保つ
- **YAGNI(You Aren't Gonna Need It)**: 必要になるまで実装しない
- **Single Responsibility Principle**: 単一責任の原則
- **Open/Closed Principle**: 開放/閉鎖原則
