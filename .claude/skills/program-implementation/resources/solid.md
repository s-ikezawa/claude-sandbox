# SOLID原則

## クイックリファレンス(実装時のチェックリスト)

SOLID原則適用時は以下をチェックしてください:

- [ ] 各クラスは単一の責務を持っているか? (S)
- [ ] 新機能追加時に既存コードを変更せず拡張できるか? (O)
- [ ] 派生クラスは基底クラスと置き換え可能か? (L)
- [ ] 使わないメソッドの実装を強制していないか? (I)
- [ ] 具体的な実装ではなく抽象に依存しているか? (D)

### TDDとの関係
- **Green段階では気にしなくてよい**: テストを通すことに集中
- **Refactor段階で段階的に適用**: すべてのテストがGreenになってから
- **一度に完璧にしない**: 継続的なリファクタリングで段階的に改善
- テストがあるため、安全にリファクタリング可能

### 他の原則との関係
- **KISS原則**: 過度な抽象化は複雑性を増す（KISS > SOLID）
- **YAGNI原則**: 必要になってから適用する
- **適用レベルの調整**: 過度な抽象化を避け、実際の要件に基づいて段階的に適用

---

## 概要

SOLID原則は、オブジェクト指向プログラミングにおける5つの設計原則です。Robert C. Martin（Uncle Bob）によって提唱され、**保守性が高く、拡張しやすく、理解しやすいソフトウェア**を設計するためのガイドラインとして、2025年現在でも広く使用されています。

---

## S: 単一責任の原則（Single Responsibility Principle）

### 定義
**「クラスは変更する理由を1つだけ持つべきである」**

1つのクラスは1つの責務のみを持ち、複数の責務を担当すべきではありません。

### 目的
- コードの変更が他の機能に影響を与えることを防ぐ
- テストとメンテナンスを容易にする

### コード例

```typescript
// ❌ 悪い例：複数の責務を持つ
class User {
  saveToDatabase() { /* DB操作 */ }
  sendEmail() { /* メール送信 */ }
  generateReport() { /* レポート生成 */ }
}

// ✅ 良い例：責務を分離
class User { /* ユーザー情報のみ */ }
class UserRepository { save(user: User) { /* DB操作 */ } }
class EmailService { send(email: string, msg: string) { /* メール送信 */ } }
class UserReportGenerator { generate(user: User) { /* レポート生成 */ } }
```

---

## O: 開放閉鎖の原則（Open/Closed Principle）

### 定義
**「拡張に対して開かれており、修正に対して閉じているべきである」**

新機能追加時、既存コードを変更せず、拡張で実現すべきです。

### 目的
- 既存コードの変更によるバグの混入を防ぐ
- 新機能追加時のリスクを最小化する

### コード例

```typescript
// ❌ 悪い例：新しい決済方法を追加するたびに修正が必要
class PaymentProcessor {
  processPayment(type: string, amount: number) {
    if (type === 'credit_card') { /* 処理 */ }
    else if (type === 'paypal') { /* 処理 */ }
    // 新しい決済方法を追加するたびにこのクラスを修正
  }
}

// ✅ 良い例：拡張で対応
interface PaymentMethod {
  process(amount: number): void;
}

class CreditCardPayment implements PaymentMethod {
  process(amount: number) { console.log(`Credit card: $${amount}`); }
}

class PayPalPayment implements PaymentMethod {
  process(amount: number) { console.log(`PayPal: $${amount}`); }
}

class PaymentProcessor {
  processPayment(method: PaymentMethod, amount: number) {
    method.process(amount); // 既存コードは変更不要
  }
}
```

---

## L: リスコフの置換原則（Liskov Substitution Principle）

### 定義
**「派生クラスは基底クラスと置き換え可能でなければならない」**

親クラスのインスタンスを子クラスに置き換えても、プログラムの正しさが保たれるべきです。

### 目的
- 継承の正しい使用を保証する
- 予期しない動作を防ぐ

### コード例

```typescript
// ❌ 悪い例：Squareは期待される動作をしない
class Rectangle {
  setWidth(w: number) { this.width = w; }
  setHeight(h: number) { this.height = h; }
  getArea() { return this.width * this.height; }
}

class Square extends Rectangle {
  setWidth(w: number) { this.width = this.height = w; } // 予期しない動作
  setHeight(h: number) { this.width = this.height = h; }
}

function test(rect: Rectangle) {
  rect.setWidth(5);
  rect.setHeight(10);
  console.log(rect.getArea()); // Rectangle: 50, Square: 100（不整合）
}

// ✅ 良い例：継承ではなくインターフェースで統一
interface Shape {
  getArea(): number;
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}
  getArea() { return this.width * this.height; }
}

class Square implements Shape {
  constructor(private side: number) {}
  getArea() { return this.side * this.side; }
}
```

---

## I: インターフェース分離の原則（Interface Segregation Principle）

### 定義
**「クライアントは使用しないメソッドへの依存を強制されるべきではない」**

大きなインターフェースより、小さく特化したインターフェースを使用すべきです。

### 目的
- 不要な依存関係を排除する
- クライアントごとに最適なインターフェースを提供する

### コード例

```typescript
// ❌ 悪い例：すべてのメソッドを実装する必要がある
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
  charge(): void;
}

class HumanWorker implements Worker {
  work() { console.log('Working'); }
  eat() { console.log('Eating'); }
  sleep() { console.log('Sleeping'); }
  charge() { throw new Error('Humans cannot charge!'); } // 不要
}

class RobotWorker implements Worker {
  work() { console.log('Working'); }
  eat() { throw new Error('Robots do not eat!'); } // 不要
  sleep() { throw new Error('Robots do not sleep!'); } // 不要
  charge() { console.log('Charging'); }
}

// ✅ 良い例：小さく特化したインターフェース
interface Workable { work(): void; }
interface Eatable { eat(): void; }
interface Sleepable { sleep(): void; }
interface Chargeable { charge(): void; }

class HumanWorker implements Workable, Eatable, Sleepable {
  work() { console.log('Working'); }
  eat() { console.log('Eating'); }
  sleep() { console.log('Sleeping'); }
}

class RobotWorker implements Workable, Chargeable {
  work() { console.log('Working'); }
  charge() { console.log('Charging'); }
}
```

---

## D: 依存性逆転の原則（Dependency Inversion Principle）

### 定義
**「高レベルモジュールは低レベルモジュールに依存してはならない。両方とも抽象に依存すべきである」**

具体的な実装ではなく、抽象（インターフェース）に依存することで柔軟性を高めます。

### 目的
- 具体的な実装の変更が他のモジュールに影響しないようにする
- テストしやすいコードを書く

### コード例

```typescript
// ❌ 悪い例：具体的な実装に依存
class MySQLDatabase {
  connect() { console.log('MySQL connected'); }
  query(sql: string) { console.log(`Query: ${sql}`); }
}

class UserService {
  private db = new MySQLDatabase(); // 具体的な実装に依存
  getUser(id: number) {
    this.db.connect();
    this.db.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}

// ✅ 良い例：抽象に依存
interface Database {
  connect(): void;
  query(sql: string): void;
}

class MySQLDatabase implements Database {
  connect() { console.log('MySQL connected'); }
  query(sql: string) { console.log(`MySQL query: ${sql}`); }
}

class PostgreSQLDatabase implements Database {
  connect() { console.log('PostgreSQL connected'); }
  query(sql: string) { console.log(`PostgreSQL query: ${sql}`); }
}

class UserService {
  constructor(private db: Database) {} // 抽象に依存
  getUser(id: number) {
    this.db.connect();
    this.db.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}

// 使用例：簡単に切り替え可能
const mysqlService = new UserService(new MySQLDatabase());
const pgService = new UserService(new PostgreSQLDatabase());
```

---

## SOLID原則のメリット

1. **保守性の向上**: コードの変更が局所的になり、影響範囲が限定される
2. **拡張性の確保**: 既存コードを変更せずに新機能を追加できる
3. **テスト容易性**: 各クラスの責務が明確で、モックやスタブを使いやすい
4. **結合度の低下**: モジュール間の依存関係が減り、独立性が高まる
5. **可読性の向上**: 各クラスの役割が明確で、コードが理解しやすい

## 注意点

- SOLID原則は**ガイドライン**であり、すべての状況に適用できるわけではない
- **過度な抽象化**は逆に複雑性を増す可能性がある
- プロジェクトの規模や要件に応じて、適切なバランスを取ることが重要
- 小規模プロジェクトやプロトタイプでは、すべての原則を厳密に適用する必要はない場合もある

## SOLID原則を適用すべきタイミング

### 適用を検討すべきサイン

以下のいずれかに該当する場合、SOLID原則の適用を検討します:

- [ ] **クラスが複数の責務を持ち始めた**（単一責任の原則）
  - クラスが100行以上になっている
  - メソッドが10個以上ある
  - 「〜と〜を行うクラス」と説明される（"and"が入る）

- [ ] **似たような処理を行うクラスが複数存在する**（開放閉鎖の原則）
  - 新機能追加時に既存コードの変更が頻繁に発生する
  - if/switchで型を判定している箇所がある

- [ ] **テストでモックを作りにくい**（依存性逆転の原則）
  - コンストラクタ内で`new`を使って具体クラスを生成している
  - 外部サービスやDBに直接依存している

- [ ] **継承階層が3階層以上になっている**（リスコフの置換原則）
  - 子クラスが親クラスのメソッドを多数オーバーライドしている
  - 継承の代わりにコンポジションを検討すべきサイン

- [ ] **インターフェースが肥大化している**（インターフェース分離の原則）
  - インターフェースに10個以上のメソッドがある
  - 実装クラスで不要なメソッドを空実装している

### まだ適用不要なケース

以下の場合は、SOLID原則を厳密に適用する必要はありません:

- ✅ **クラスが1つしかない**
  - まだ抽象化の必要性が見えていない段階
  - YAGNI原則に従い、必要になってから適用

- ✅ **メソッドが5個以下でシンプル**
  - 責務が明確で理解しやすい
  - 無理に分割すると逆に複雑になる

- ✅ **プロトタイプや短命なスクリプト**
  - 一度きりの使い捨てコード
  - KISS原則を優先

- ✅ **要件が固まっていない初期段階**
  - 過度な抽象化は変更を困難にする
  - まずはシンプルに実装し、パターンが見えてから適用

### TDDとの関係

- **Green段階**: SOLID原則は気にしない（テストを通すことに集中）
- **Refactor段階**: 上記のサインが見られたら、段階的に適用
- **一度に完璧にしない**: 継続的なリファクタリングで段階的に改善

### 優先順位

迷った場合の優先順位:

1. **KISS原則**: まずはシンプルに（過度な抽象化を避ける）
2. **YAGNI原則**: 必要になってから適用
3. **SOLID原則**: 上記のサインが見られたら段階的に適用

**原則**: 「シンプルさ」と「拡張性」のバランスを取る。複雑性が現れてから対処する。

## まとめ

SOLID原則を理解し適切に適用することで、変更に強く、拡張しやすく、メンテナンスしやすいコードを書くことができます。各原則の頭文字を覚えるだけでなく、**その背後にある「なぜ」を理解すること**が重要です。
