# デメテルの法則 (Law of Demeter)

## クイックリファレンス(実装時のチェックリスト)

デメテルの法則適用時は以下をチェックしてください:

- [ ] メソッドチェーン（`a.getB().getC()`）を使っていないか?
- [ ] 直接知らないオブジェクトのメソッドを呼んでいないか?
- [ ] 「.」が2つ以上連続していないか?
- [ ] オブジェクトの内部構造に依存していないか?
- [ ] カプセル化を破壊していないか?

### TDDとの関係
- **Green段階では気にしなくてよい**: テストを通すことに集中
- **Refactor段階で適用**: すべてのテストがGreenになってから
- **テストしやすい設計**: デメテルの法則に従うとモックやスタブが使いやすい

### 他の原則との関係
- **SOLID-D（依存性逆転）**: 具体的な実装の詳細に依存しない（補完関係）
- **SOLID-S（単一責任）**: 各オブジェクトが適切な責務を持つ（調和）
- **カプセル化の強化**: オブジェクトの内部実装を隠蔽

---

## 概要

デメテルの法則（Law of Demeter）は、別名「最小知識の原則（Principle of Least Knowledge）」とも呼ばれ、**オブジェクトは直接の友人とだけ話すべき**という設計原則です。

### 核心的な考え方

> "Each unit should only talk to its friends; don't talk to strangers."
>
> 各ユニットは直接の友人とだけ話すべきで、見知らぬ人と話してはならない。

> "Only use one dot."
>
> ドットは1つだけ使う。（簡略化されたルール）

## デメテルの法則の重要性

### メリット

1. **低い結合度**: オブジェクト間の依存関係が減る
2. **高い保守性**: 内部実装の変更が他に影響しにくい
3. **再利用性の向上**: クラスが独立しているため再利用しやすい
4. **テストの容易性**: モックやスタブが作りやすい
5. **カプセル化の強化**: オブジェクトの内部が隠蔽される
6. **バグの削減**: 変更の影響範囲が限定される

### デメリット(過度な適用の場合)

1. **ラッパーメソッドの増加**: 委譲メソッドが大量に必要になる場合がある
2. **パフォーマンス**: 委譲の層が増えることでわずかなオーバーヘッド

## デメテルの法則の詳細ルール

### メソッドが呼び出せるオブジェクト

あるメソッド`m`は、以下のオブジェクトのメソッドのみ呼び出すべきです:

1. **自分自身**（`this`）
2. **mの引数**
3. **m内で作成したオブジェクト**
4. **自分のインスタンス変数**
5. **グローバル変数**

### 呼び出してはいけないもの

- **他のメソッドから返されたオブジェクト**のメソッド

## デメテルの法則違反のパターンと改善例

### パターン1: メソッドチェーン

#### ❌ 悪い例

```typescript
// 顧客の住所の郵便番号を取得
class Order {
  getCustomerZipCode(): string {
    // 違反: customer → address → zipCode と辿っている
    return this.customer.getAddress().getZipCode();
  }
}

// 使用例
const zipCode = order.getCustomer().getAddress().getZipCode();
```

**問題点:**
- `Order`が`Customer`の内部構造（`Address`を持つこと）を知っている
- `Customer`が`Address`の内部構造（`zipCode`を持つこと）を知っている
- `Address`の実装変更が`Order`に影響する

#### ✅ 良い例

```typescript
// Customerに委譲メソッドを追加
class Customer {
  getZipCode(): string {
    return this.address.getZipCode();
  }
}

// Orderはシンプルに
class Order {
  getCustomerZipCode(): string {
    return this.customer.getZipCode();
  }
}

// 使用例
const zipCode = order.getCustomerZipCode();
```

**改善点:**
- `Order`は`Customer`の内部構造を知らない
- `Address`の実装変更は`Customer`内で吸収される
- 各オブジェクトが適切にカプセル化されている

---

### パターン2: 深いネスト構造のナビゲーション

#### ❌ 悪い例

```typescript
class Company {
  getDepartmentManagerEmail(deptName: string): string {
    // 違反: company → department → manager → contact → email
    return this.departments
      .find(d => d.getName() === deptName)
      .getManager()
      .getContactInfo()
      .getEmail();
  }
}
```

#### ✅ 良い例

```typescript
// Department に委譲
class Department {
  getManagerEmail(): string {
    return this.manager.getEmail();
  }
}

// Manager に委譲
class Manager {
  getEmail(): string {
    return this.contactInfo.getEmail();
  }
}

// Company はシンプルに
class Company {
  getDepartmentManagerEmail(deptName: string): string {
    const department = this.findDepartment(deptName);
    return department.getManagerEmail();
  }

  private findDepartment(name: string): Department {
    return this.departments.find(d => d.getName() === name);
  }
}
```

---

### パターン3: DTOやデータ構造との混同

#### ❌ 違反ではない例（データ構造）

```typescript
// DTOやデータ構造の場合は例外
interface Point {
  x: number;
  y: number;
}

interface Rectangle {
  topLeft: Point;
  bottomRight: Point;
}

// これは許容される（データ構造なので）
function calculateArea(rect: Rectangle): number {
  const width = rect.bottomRight.x - rect.topLeft.x;
  const height = rect.bottomRight.y - rect.topLeft.y;
  return width * height;
}
```

**注意**: デメテルの法則は**オブジェクト**に適用されます。単なるデータ構造（振る舞いを持たないDTO）には適用されません。

#### ✅ オブジェクトの場合

```typescript
// オブジェクトとして設計する場合
class Point {
  constructor(private x: number, private y: number) {}

  getX(): number { return this.x; }
  getY(): number { return this.y; }
}

class Rectangle {
  constructor(
    private topLeft: Point,
    private bottomRight: Point
  ) {}

  // デメテルの法則に従う
  getArea(): number {
    const width = this.bottomRight.getX() - this.topLeft.getX();
    const height = this.bottomRight.getY() - this.topLeft.getY();
    return width * height;
  }
}

// 使用例
const rect = new Rectangle(new Point(0, 0), new Point(10, 10));
const area = rect.getArea();  // カプセル化されている
```

---

### パターン4: ビルダーパターンとの混同

#### ❌ 違反ではない例（Fluent Interface / メソッドチェーン）

```typescript
// Fluent Interfaceは例外（自分自身を返すパターン）
class QueryBuilder {
  private conditions: string[] = [];

  where(condition: string): QueryBuilder {
    this.conditions.push(condition);
    return this;  // 自分自身を返す
  }

  orderBy(field: string): QueryBuilder {
    // ...
    return this;  // 自分自身を返す
  }

  build(): string {
    // クエリ文字列を生成
    return `SELECT * WHERE ${this.conditions.join(' AND ')}`;
  }
}

// これは許容される（すべて同じオブジェクト）
const query = new QueryBuilder()
  .where('age > 18')
  .where('active = true')
  .orderBy('name')
  .build();
```

**理由**: すべてのメソッドが同じオブジェクト（`this`）を返しているため、デメテルの法則に違反していません。

---

### パターン5: 外部ライブラリのAPI

#### ❌ 悪い例

```typescript
// DOMの深いナビゲーション
function updateTitle() {
  document
    .getElementById('app')
    .querySelector('.header')
    .querySelector('.title')
    .textContent = 'New Title';
}
```

#### ✅ 良い例

```typescript
// 直接ターゲットを取得
function updateTitle() {
  const titleElement = document.querySelector('#app .header .title');
  if (titleElement) {
    titleElement.textContent = 'New Title';
  }
}

// またはヘルパー関数で隠蔽
class DOMHelper {
  static updateTitle(text: string) {
    const titleElement = document.querySelector('#app .header .title');
    if (titleElement) {
      titleElement.textContent = text;
    }
  }
}

// 使用
DOMHelper.updateTitle('New Title');
```

---

## デメテルの法則適用のベストプラクティス

### 1. Tell, Don't Ask（命令せよ、尋ねるな）

```typescript
// ❌ Ask（尋ねる）スタイル
if (user.getWallet().getBalance() > 100) {
  user.getWallet().deduct(100);
}

// ✅ Tell（命令する）スタイル
user.purchase(100);

// User クラス内部
class User {
  purchase(amount: number): boolean {
    if (this.wallet.hasEnough(amount)) {
      this.wallet.deduct(amount);
      return true;
    }
    return false;
  }
}
```

### 2. 委譲メソッドの追加

```typescript
// ❌ 内部構造に依存
class Order {
  processOrder() {
    const email = this.customer.getContactInfo().getEmail();
    sendEmail(email, 'Order processed');
  }
}

// ✅ 委譲メソッドを追加
class Customer {
  getEmail(): string {
    return this.contactInfo.getEmail();
  }
}

class Order {
  processOrder() {
    const email = this.customer.getEmail();
    sendEmail(email, 'Order processed');
  }
}
```

### 3. インターフェースの抽出

```typescript
// ❌ 具体的な実装に依存
class PaymentProcessor {
  process(order: Order) {
    const amount = order.getCart().getItems().reduce(
      (sum, item) => sum + item.getPrice(),
      0
    );
    // 支払い処理
  }
}

// ✅ インターフェースで抽象化
interface Payable {
  getTotalAmount(): number;
}

class Order implements Payable {
  getTotalAmount(): number {
    return this.cart.getTotalAmount();
  }
}

class Cart {
  getTotalAmount(): number {
    return this.items.reduce((sum, item) => sum + item.getPrice(), 0);
  }
}

class PaymentProcessor {
  process(payable: Payable) {
    const amount = payable.getTotalAmount();
    // 支払い処理
  }
}
```

### 4. 適切な責務の配置

```typescript
// ❌ 不適切な責務の配置
class ReportGenerator {
  generate(user: User): string {
    // User の内部構造に深く依存
    const orders = user.getOrderHistory().getOrders();
    const total = orders.reduce(
      (sum, order) => sum + order.getTotal(),
      0
    );
    return `Total: ${total}`;
  }
}

// ✅ 適切な責務の配置
class User {
  getTotalOrderAmount(): number {
    return this.orderHistory.getTotalAmount();
  }
}

class OrderHistory {
  getTotalAmount(): number {
    return this.orders.reduce(
      (sum, order) => sum + order.getTotal(),
      0
    );
  }
}

class ReportGenerator {
  generate(user: User): string {
    const total = user.getTotalOrderAmount();
    return `Total: ${total}`;
  }
}
```

## TDDとデメテルの法則

### Refactor段階での適用

```typescript
// Red & Green段階: まず動くコードを書く
test('注文の合計金額を計算できる', () => {
  const order = new Order(customer, cart);
  expect(order.getTotalPrice()).toBe(1000);
});

// Green段階の実装（デメテルの法則を無視してもOK）
class Order {
  getTotalPrice(): number {
    return this.cart.getItems().reduce(
      (sum, item) => sum + item.getPrice(),
      0
    );
  }
}

// Refactor段階: デメテルの法則を適用
class Cart {
  getTotalPrice(): number {
    return this.items.reduce(
      (sum, item) => sum + item.getPrice(),
      0
    );
  }
}

class Order {
  getTotalPrice(): number {
    return this.cart.getTotalPrice();  // デメテルの法則に従う
  }
}
```

## デメテルの法則を適用すべき/すべきでない状況

### 適用すべき状況

1. **オブジェクト指向設計**
   ```typescript
   // ビジネスロジックを持つオブジェクト
   class User {
     purchaseItem(item: Item) {
       // デメテルの法則を適用
     }
   }
   ```

2. **長期的に保守するコード**
   - 内部実装の変更が頻繁に発生する場合

3. **複数チームでの開発**
   - カプセル化が重要な場合

### 適用しなくてよい状況

1. **データ構造やDTO**
   ```typescript
   interface UserDTO {
     name: string;
     email: string;
   }
   // DTOはデータの入れ物なのでOK
   const email = userDTO.email;
   ```

2. **Fluent Interface**
   ```typescript
   // ビルダーパターンなど
   query.where().orderBy().limit();
   ```

3. **フレームワークのAPI**
   ```typescript
   // ライブラリやフレームワークが提供するAPI
   express().use().listen();
   ```

4. **短命なスクリプト**
   - ワンタイムスクリプトなど

## デメテルの法則のチェックリスト

実装時に以下を確認してください:

- [ ] メソッドチェーン（連続した`.`）を使っていないか?
- [ ] 他のオブジェクトから返されたオブジェクトのメソッドを呼んでいないか?
- [ ] オブジェクトの内部構造に依存していないか?
- [ ] "Tell, Don't Ask" の原則に従っているか?
- [ ] 委譲メソッドを適切に配置しているか?

リファクタリング時に以下を確認してください:

- [ ] 委譲メソッドが過剰に増えていないか?
- [ ] 本当にオブジェクトか、それとも単なるデータ構造か?
- [ ] Fluent Interfaceなど、例外的なパターンではないか?

## まとめ

### デメテルの法則の本質

1. **直接の友人とだけ話す** - 見知らぬ人とは話さない
2. **カプセル化を保つ** - オブジェクトの内部構造を隠蔽する
3. **低い結合度** - オブジェクト間の依存関係を減らす
4. **Tell, Don't Ask** - 尋ねずに命令する

### 心に留めておくべきこと

1. **ドットは1つまで** - 簡略化されたルールとして有用
2. **データ構造は例外** - オブジェクトとデータ構造を区別する
3. **過度な委譲に注意** - バランスが重要
4. **Refactor段階で適用** - TDDのGreen段階では無視してもOK

### 関連する原則

- **SOLID-D（依存性逆転）**: 抽象に依存し、具体的な実装の詳細に依存しない
- **SOLID-S（単一責任）**: 各オブジェクトが適切な責務を持つ
- **カプセル化**: オブジェクトの内部実装を隠蔽
- **Tell, Don't Ask**: 命令型のインターフェース設計
