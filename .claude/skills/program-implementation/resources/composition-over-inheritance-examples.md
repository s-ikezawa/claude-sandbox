# コンポジション優先原則 - 詳細なコード例

このファイルには、[コンポジション優先原則](composition-over-inheritance.md)の詳細なコード例を掲載しています。

## 目次
- [パターン1: コードの再利用のための継承](#パターン1-コードの再利用のための継承)
- [パターン2: 複雑な継承階層](#パターン2-複雑な継承階層)
- [パターン3: ダイヤモンド問題](#パターン3-ダイヤモンド問題)
- [パターン4: 脆弱な基底クラス問題](#パターン4-脆弱な基底クラス問題)
- [ベストプラクティス](#ベストプラクティス)

---

## パターン1: コードの再利用のための継承

### ❌ 悪い例（継承の誤用）

```typescript
// Stack は ArrayList ではない（is-a 関係にない）
class ArrayList<T> {
  private items: T[] = [];

  add(item: T) {
    this.items.push(item);
  }

  get(index: number): T {
    return this.items[index];
  }

  remove(index: number) {
    this.items.splice(index, 1);
  }

  size(): number {
    return this.items.length;
  }
}

// 継承によるコード再利用（間違い）
class Stack<T> extends ArrayList<T> {
  push(item: T) {
    this.add(item);
  }

  pop(): T | undefined {
    const item = this.get(this.size() - 1);
    this.remove(this.size() - 1);
    return item;
  }
}

// 問題: Stack に不要なメソッドが公開される
const stack = new Stack<number>();
stack.push(1);
stack.push(2);
stack.get(0);  // スタックの途中にアクセスできてしまう!
stack.remove(0);  // スタックの途中を削除できてしまう!
```

### ✅ 良い例（コンポジション）

```typescript
// Stack has an ArrayList
class Stack<T> {
  private items: ArrayList<T> = new ArrayList();

  push(item: T) {
    this.items.add(item);
  }

  pop(): T | undefined {
    if (this.items.size() === 0) return undefined;
    const item = this.items.get(this.items.size() - 1);
    this.items.remove(this.items.size() - 1);
    return item;
  }

  peek(): T | undefined {
    if (this.items.size() === 0) return undefined;
    return this.items.get(this.items.size() - 1);
  }

  size(): number {
    return this.items.size();
  }
}

// Stackとして必要なメソッドだけが公開される
const stack = new Stack<number>();
stack.push(1);
stack.push(2);
// stack.get(0);  // エラー: このメソッドは存在しない（正しい）
```

---

## パターン2: 複雑な継承階層

### ❌ 悪い例（深い継承階層）

```typescript
class Vehicle {
  move() { /* ... */ }
}

class LandVehicle extends Vehicle {
  drive() { /* ... */ }
}

class Car extends LandVehicle {
  openDoors() { /* ... */ }
}

class ElectricCar extends Car {
  charge() { /* ... */ }
}

class TeslaModelS extends ElectricCar {
  autopilot() { /* ... */ }
}

// 問題:
// - 5階層の深い継承
// - 中間層の変更が下層に影響
// - テストが困難
```

### ✅ 良い例（コンポジション）

```typescript
// 振る舞いをインターフェースで定義
interface Movable {
  move(): void;
}

interface Drivable {
  drive(): void;
}

interface Chargeable {
  charge(): void;
}

// 具体的な機能をコンポーネントとして実装
class ElectricEngine {
  start() { /* ... */ }
  charge() { /* ... */ }
}

class AutopilotSystem {
  engage() { /* ... */ }
}

class DoorSystem {
  open() { /* ... */ }
  close() { /* ... */ }
}

// コンポジションで組み合わせる
class TeslaModelS implements Movable, Drivable, Chargeable {
  private engine: ElectricEngine;
  private autopilot: AutopilotSystem;
  private doors: DoorSystem;

  constructor() {
    this.engine = new ElectricEngine();
    this.autopilot = new AutopilotSystem();
    this.doors = new DoorSystem();
  }

  move() {
    this.engine.start();
  }

  drive() {
    // 運転ロジック
  }

  charge() {
    this.engine.charge();
  }

  enableAutopilot() {
    this.autopilot.engage();
  }

  openDoors() {
    this.doors.open();
  }
}

// メリット:
// - フラットな構造
// - 各コンポーネントが独立してテスト可能
// - 実行時に振る舞いを変更可能
```

---

## パターン3: ダイヤモンド問題（多重継承の問題）

### ❌ 悪い例（多重継承が必要になる設計）

```typescript
// TypeScriptは多重継承をサポートしていないが、概念的な例

// 飛べる鳥
class FlyingBird extends Bird {
  fly() { /* ... */ }
}

// 泳げる鳥
class SwimmingBird extends Bird {
  swim() { /* ... */ }
}

// ペンギンは泳げるが飛べない
// class Penguin extends SwimmingBird {
//   // OK: 泳げる
// }

// カモは飛べるし泳げる
// class Duck extends ??? {
//   // 問題: FlyingBirdとSwimmingBirdの両方を継承したい
// }
```

### ✅ 良い例（コンポジション）

```typescript
// 振る舞いをコンポーネントとして分離
interface Flyable {
  fly(): void;
}

interface Swimmable {
  swim(): void;
}

class FlyingAbility implements Flyable {
  fly() {
    console.log('Flying in the sky');
  }
}

class SwimmingAbility implements Swimmable {
  swim() {
    console.log('Swimming in water');
  }
}

// ペンギン: 泳げるが飛べない
class Penguin implements Swimmable {
  private swimmingAbility: SwimmingAbility;

  constructor() {
    this.swimmingAbility = new SwimmingAbility();
  }

  swim() {
    this.swimmingAbility.swim();
  }
}

// カモ: 飛べるし泳げる
class Duck implements Flyable, Swimmable {
  private flyingAbility: FlyingAbility;
  private swimmingAbility: SwimmingAbility;

  constructor() {
    this.flyingAbility = new FlyingAbility();
    this.swimmingAbility = new SwimmingAbility();
  }

  fly() {
    this.flyingAbility.fly();
  }

  swim() {
    this.swimmingAbility.swim();
  }
}

// ダチョウ: 飛べないし泳げない（能力を持たない）
class Ostrich {
  run() {
    console.log('Running fast');
  }
}
```

---

## パターン4: 脆弱な基底クラス問題

### ❌ 悪い例（基底クラスの変更が子クラスを破壊）

```typescript
// 基底クラス
class Counter {
  private count = 0;

  increment() {
    this.count++;
  }

  getCount(): number {
    return this.count;
  }
}

// 子クラス
class LimitedCounter extends Counter {
  constructor(private limit: number) {
    super();
  }

  // incrementをオーバーライド
  increment() {
    if (this.getCount() < this.limit) {
      super.increment();
    }
  }
}

// 後で基底クラスが変更される...
class Counter {
  private count = 0;

  increment() {
    this.add(1);  // 実装を変更
  }

  add(value: number) {
    this.count += value;
  }

  getCount(): number {
    return this.count;
  }
}

// 問題: LimitedCounterが壊れる可能性がある
```

### ✅ 良い例（コンポジション）

```typescript
// Counterを変更不可能なコンポーネントとして扱う
class Counter {
  private count = 0;

  increment() {
    this.count++;
  }

  getCount(): number {
    return this.count;
  }
}

// コンポジションで機能を拡張
class LimitedCounter {
  private counter: Counter = new Counter();

  constructor(private limit: number) {}

  increment() {
    if (this.counter.getCount() < this.limit) {
      this.counter.increment();
    }
  }

  getCount(): number {
    return this.counter.getCount();
  }
}

// Counterの内部実装が変わってもLimitedCounterは影響を受けにくい
```

---

## ベストプラクティス

### 1. インターフェースの活用

```typescript
// インターフェースで契約を定義
interface Logger {
  log(message: string): void;
}

class ConsoleLogger implements Logger {
  log(message: string) {
    console.log(message);
  }
}

class FileLogger implements Logger {
  log(message: string) {
    // ファイルに書き込み
  }
}

// コンポジションで柔軟に組み合わせ
class UserService {
  constructor(private logger: Logger) {}

  createUser(name: string) {
    this.logger.log(`Creating user: ${name}`);
    // ユーザー作成処理
  }
}

// 実行時に Logger の実装を切り替え可能
const service1 = new UserService(new ConsoleLogger());
const service2 = new UserService(new FileLogger());
```

### 2. 依存性の注入（DI）

```typescript
// コンポーネントを外部から注入
class EmailSender {
  send(to: string, message: string) {
    // メール送信
  }
}

class NotificationService {
  // コンストラクタで依存を注入
  constructor(private emailSender: EmailSender) {}

  notify(user: User, message: string) {
    this.emailSender.send(user.email, message);
  }
}

// テスト時にモックを注入可能
class MockEmailSender {
  send(to: string, message: string) {
    console.log(`Mock: Sending email to ${to}`);
  }
}

const testService = new NotificationService(new MockEmailSender());
```

### 3. Strategy パターン

```typescript
// 戦略をインターフェースで定義
interface PaymentStrategy {
  pay(amount: number): void;
}

class CreditCardPayment implements PaymentStrategy {
  pay(amount: number) {
    console.log(`Paid ${amount} by credit card`);
  }
}

class PayPalPayment implements PaymentStrategy {
  pay(amount: number) {
    console.log(`Paid ${amount} by PayPal`);
  }
}

// コンテキスト
class ShoppingCart {
  private items: Item[] = [];
  private paymentStrategy: PaymentStrategy;

  setPaymentStrategy(strategy: PaymentStrategy) {
    this.paymentStrategy = strategy;
  }

  checkout() {
    const total = this.calculateTotal();
    this.paymentStrategy.pay(total);
  }

  private calculateTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
}

// 実行時に戦略を変更可能
const cart = new ShoppingCart();
cart.setPaymentStrategy(new CreditCardPayment());
cart.checkout();

cart.setPaymentStrategy(new PayPalPayment());
cart.checkout();
```

### 4. Decorator パターン

```typescript
// 基本のコンポーネント
interface Coffee {
  cost(): number;
  description(): string;
}

class SimpleCoffee implements Coffee {
  cost() {
    return 100;
  }

  description() {
    return 'Simple coffee';
  }
}

// デコレータ
class MilkDecorator implements Coffee {
  constructor(private coffee: Coffee) {}

  cost() {
    return this.coffee.cost() + 50;
  }

  description() {
    return this.coffee.description() + ', milk';
  }
}

class SugarDecorator implements Coffee {
  constructor(private coffee: Coffee) {}

  cost() {
    return this.coffee.cost() + 20;
  }

  description() {
    return this.coffee.description() + ', sugar';
  }
}

// 柔軟な組み合わせ
let coffee: Coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);

console.log(coffee.description());  // "Simple coffee, milk, sugar"
console.log(coffee.cost());  // 170
```

---

さらに詳しい説明は[コンポジション優先原則](composition-over-inheritance.md)を参照してください。
