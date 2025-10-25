# コンポジション優先原則 (Composition over Inheritance)

## クイックリファレンス(実装時のチェックリスト)

コンポジション優先原則適用時は以下をチェックしてください:

- [ ] 継承を使おうとしているが、本当に is-a 関係か?
- [ ] has-a 関係で表現できないか?
- [ ] 継承階層が3階層以上になっていないか?
- [ ] 多重継承や複雑な継承構造になっていないか?
- [ ] 振る舞いを再利用したいだけではないか?（それならコンポジション）

### TDDとの関係
- **Green段階では気にしなくてよい**: テストを通すことに集中
- **Refactor段階で検討**: すべてのテストがGreenになってから、設計を見直す
- **テストしやすさ**: コンポジションの方がモックやスタブを使いやすい

### 他の原則との関係
- **SOLID-L（リスコフの置換原則）**: 継承の誤用による問題を防ぐ
- **SOLID-D（依存性逆転）**: インターフェースを通じた依存が容易（調和）
- **柔軟性の向上**: 実行時に振る舞いを変更可能

---

## 概要

コンポジション優先原則（Composition over Inheritance）は、**継承（Inheritance）よりもコンポジション（Composition）を優先すべき**という設計原則です。Gang of Four（GoF）の『デザインパターン』でも強調されています。

### 核心的な考え方

> "Favor object composition over class inheritance."
>
> クラス継承よりもオブジェクトのコンポジションを優先しなさい。

## 用語の定義

### 継承 (Inheritance)
- **is-a 関係**: "A is a B"（AはBである）
- サブクラスがスーパークラスを拡張する
- 実装の再利用

```typescript
// Dog is an Animal
class Animal {
  eat() { /* ... */ }
}

class Dog extends Animal {
  bark() { /* ... */ }
}
```

### コンポジション (Composition)
- **has-a 関係**: "A has a B"（AはBを持つ）
- オブジェクトが他のオブジェクトを含む
- 機能の委譲

```typescript
// Car has an Engine
class Engine {
  start() { /* ... */ }
}

class Car {
  private engine: Engine;

  constructor() {
    this.engine = new Engine();
  }

  start() {
    this.engine.start();
  }
}
```

## コンポジション優先原則の重要性

### 継承の問題点

1. **強い結合**: 親クラスと子クラスが密結合
2. **脆弱な基底クラス問題**: 親クラスの変更が子クラスに影響
3. **柔軟性の欠如**: 実行時に振る舞いを変更できない
4. **多重継承の問題**: 多くの言語で多重継承は制限されている
5. **カプセル化の破壊**: 子クラスが親クラスの内部実装に依存しがち

### コンポジションのメリット

1. **低い結合度**: オブジェクト間が疎結合
2. **高い柔軟性**: 実行時に振る舞いを変更可能
3. **再利用性**: 小さなコンポーネントを組み合わせて再利用
4. **テストの容易性**: モックやスタブが使いやすい
5. **カプセル化の保持**: 内部実装が隠蔽される

## 基本的なコード例

### 継承の誤用（コードの再利用のため）

```typescript
// ❌ 悪い例: Stack is-a ArrayList ではない
class Stack<T> extends ArrayList<T> {
  push(item: T) { this.add(item); }
  pop(): T { /* ... */ }
}

// 問題: 不要なメソッドが公開される
stack.get(0);  // スタックの途中にアクセスできてしまう!

// ✅ 良い例: Stack has-a ArrayList
class Stack<T> {
  private items: ArrayList<T> = new ArrayList();

  push(item: T) { this.items.add(item); }
  pop(): T { /* ... */ }
  // 必要なメソッドだけ公開
}
```

### 多重継承の問題（ダイヤモンド問題）

```typescript
// ❌ 悪い例: カモは飛べるし泳げる（多重継承したい）
// class Duck extends FlyingBird, SwimmingBird { }  // できない!

// ✅ 良い例: コンポジション
class Duck implements Flyable, Swimmable {
  private flyingAbility = new FlyingAbility();
  private swimmingAbility = new SwimmingAbility();

  fly() { this.flyingAbility.fly(); }
  swim() { this.swimmingAbility.swim(); }
}
```

## TDDとコンポジション優先原則

### Refactor段階での適用

```typescript
// Green段階: まず継承で実装してテストを通す
class Dog extends Animal {
  bark() { /* ... */ }
}

// Refactor段階: 必要に応じてコンポジションに変更
class Dog {
  private eatingBehavior = new EatingBehavior();

  eat() { this.eatingBehavior.eat(); }
  bark() { /* ... */ }
}
```

### テストの容易性

```typescript
// コンポジションはモックが使いやすい
class UserService {
  constructor(private db: Database) {}  // 注入可能
}

// テスト
const mockDb = new MockDatabase();
const service = new UserService(mockDb);  // 簡単にモック化
```

## 継承を使うべき/使うべきでない状況

### 継承を使うべき状況

1. **明確な is-a 関係がある場合**
   ```typescript
   class Dog extends Animal { }  // Dog is an Animal
   ```

2. **ポリモーフィズムが必要な場合**
   ```typescript
   abstract class Shape {
     abstract area(): number;
   }
   ```

3. **フレームワークが要求する場合**
   ```typescript
   class MyComponent extends React.Component { }
   ```

### コンポジションを使うべき状況

1. **振る舞いの再利用が目的の場合**
2. **実行時に振る舞いを変更したい場合**
3. **複数の振る舞いを組み合わせたい場合**
4. **テストしやすさが重要な場合**

## コンポジション優先原則のチェックリスト

設計時に以下を確認してください:

- [ ] 本当に is-a 関係か? has-a 関係ではないか?
- [ ] コードの再利用のためだけに継承を使っていないか?
- [ ] 継承階層が3階層以上になっていないか?
- [ ] 実行時に振る舞いを変更する必要はないか?
- [ ] 複数の親クラスから継承したいと思っていないか?
- [ ] テストでモックやスタブを使いやすいか?

リファクタリング時に以下を確認してください:

- [ ] 子クラスが親クラスのメソッドをほとんどオーバーライドしていないか?
- [ ] 親クラスの変更が子クラスに悪影響を与えていないか?
- [ ] 継承よりもコンポジションで表現した方が明確にならないか?

## 詳細なコード例

より詳細なコード例とパターンについては、以下を参照してください:
- [コンポジション優先原則 - 詳細なコード例](composition-over-inheritance-examples.md)

## まとめ

### コンポジション優先原則の本質

1. **has-a を優先** - is-a よりも has-a で考える
2. **柔軟性の確保** - 実行時に振る舞いを変更可能
3. **低い結合度** - オブジェクト間が疎結合
4. **テストの容易性** - モック・スタブが使いやすい

### 心に留めておくべきこと

1. **継承を完全に避けるわけではない** - 適切な is-a 関係なら継承も有効
2. **"優先"であって"絶対"ではない** - 状況に応じて判断する
3. **フレームワークの要求** - フレームワークが継承を要求する場合は従う
4. **Refactor段階で検討** - TDDのGreen段階では後回しでもOK

### 関連する原則

- **SOLID-L（リスコフの置換原則）**: 継承の正しい使い方を定義
- **SOLID-D（依存性逆転）**: インターフェースを通じた依存
- **Strategy パターン**: コンポジションの典型的な活用例
- **Decorator パターン**: 継承の代わりにコンポジションで機能を拡張
