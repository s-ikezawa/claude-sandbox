# テスト駆動開発(TDD)の原則

## クイックリファレンス(実装時のチェックリスト)

TDD実装時は以下を順守してください:

- [ ] 実装前に失敗するテストを書いたか? (Red)
- [ ] テストは1つの振る舞いに焦点を当てているか?
- [ ] 最小限の実装でテストを通したか? (Green)
- [ ] **現在のテストケース(1つ)がGreenになったか?**
- [ ] **既存のテストもすべてGreenのまま（リグレッションなし）か?**
- [ ] **Refactor判断チェックリストを確認したか?**
  - 重複コード3回以上（ビジネスロジックは2回）、マジックナンバー、関数が長い、ネストが深いなど
- [ ] Refactorが不要なら、次のテストケースに進んだか?
- [ ] Refactorが必要な場合、DRY/SOLID原則を適用したか?

**重要**: Green段階では重複コードや設計の粗さは許容されます。Refactorは必要になってから行います。

---

## TDDと他の原則の関係

- **Green段階**: テストを通すことに集中。重複や設計の粗さは許容する
- **Refactor段階**: DRY原則を適用して重複を除去し、SOLID原則で設計を改善
- テストがあるため、後からでも安全にリファクタリング可能

---

## 概要

テスト駆動開発(Test-Driven Development, TDD)は、テストを先に書き、そのテストを通過する最小限のコードを実装し、その後リファクタリングを行うという開発手法です。

## TDDの3つの原則

### 1. Red(失敗するテストを書く)
まず失敗するテストを書きます。この時点では実装コードがないため、テストは必ず失敗します。

### 2. Green(テストを通過する最小限のコードを書く)
テストを通過させるための最小限のコードを実装します。綺麗なコードである必要はなく、とにかくテストを通すことが目的です。

#### Green段階で許容されるもの

以下は**すべて許容されます**。Refactor段階で改善するため、Green段階では気にしないでください:

**✅ 許容される重複**:
- 同じコードが2〜3回出現する
- 似たようなロジックが複数箇所にある
- マジックナンバーやハードコードされた値

**✅ 許容される設計の粗さ**:
- 長い関数（20行以上でもOK）
- ネストが深い（3階層以上でもOK）
- メソッドチェーン（ドットが複数連続）
- 単一のクラスに複数の責務がある
- 変数名が`tmp`、`result`など曖昧
- インターフェースなしで具体クラスに直接依存

**✅ 許容されるハードコーディング**:
- `return 5;`のようなベタ書き
- 条件分岐なしの固定値返却
- テストケース固有の値の埋め込み

#### Green段階でも避けるべきもの

以下は、**次のテストケースで明らかに問題になる**ため、Green段階でも避けてください:

**❌ 避けるべきこと**:
- テストを通すために実装を完全にスキップする（空の実装）
  ```typescript
  // テストケース: add(2, 3) が 5 を返す
  function add(a: number, b: number): number {
    return 5; // テストは通るが、他の値では壊れる
  }
  ```
- 明らかに間違ったロジック（テストが偶然通っているだけ）
- 次のテストケースで確実に壊れる設計
  ```typescript
  // テストケース: 配列の最初の要素を返す
  function getFirst(arr: number[]): number {
    return arr[0]; // 空配列で次のテストが壊れる
  }

  // 次のテストケース:
  test('空配列の場合はundefinedを返す', () => {
    expect(getFirst([])).toBeUndefined(); // ← これで壊れる
  });
  ```
- セキュリティ上の問題（パスワードを平文で保存など）

#### 判断基準

**迷った時の質問**:
1. **次のテストケースで壊れるか？**
   - Yes → 修正する
   - No → Green段階では許容、Refactorで改善

2. **セキュリティ上の問題があるか？**
   - Yes → Green段階でも修正
   - No → Refactorで改善

3. **テストが偶然通っているだけか？**
   - Yes → 正しいロジックに修正
   - No → Refactorで改善

**原則**: テストを通すための最小限のコードを書く。美しさや完璧さは後回し。

#### 「次のテストケースで壊れるか」の判断方法

Green段階で実装したコードが次のテストケースで壊れるかどうか、判断が難しい場合の指針:

**ステップ1: テストケースリストを確認**
- リストアップしたテストケースの次のテストを見る
- そのテストが現在の実装で通るか想像する

**ステップ2: 判断**
- **明らかに通らない** → 今のうちに修正（Green段階でも修正すべき）
- **通る可能性がある** → Green段階では許容、次のテストが失敗したらその時に修正

**例:**
```typescript
// 現在のテストケース: add(2, 3) が 5 を返す
test('2つの数値を足し算できる', () => {
  expect(add(2, 3)).toBe(5);
});

// Green段階の実装例1（NG）
function add(a: number, b: number): number {
  return 5;  // ❌ 次のテストで確実に壊れる
}
// 次にテストするケース: add(1, 1) が 2 を返す
// → 明らかに壊れるので、Green段階でも修正すべき

// Green段階の実装例2（OK）
function add(a: number, b: number): number {
  return a + b;  // ✅ 正しい実装
}
```

**例外的なケース:**
```typescript
// 現在のテストケース: 正の数同士の足し算
test('正の数同士を足し算できる', () => {
  expect(add(2, 3)).toBe(5);
});

// Green段階の実装
function add(a: number, b: number): number {
  // 負の数のチェックは入れていない
  return a + b;
}
// 次のテストケース: add(-1, 5) が 4 を返す
// → このテストも通る可能性が高いので、Green段階では許容
```

### 3. Refactor(リファクタリング)
テストが通った状態で、コードを改善します。テストがあるため、安全にリファクタリングできます。

**この段階で行うこと**:
- **DRY原則の適用**: 重複コードを除去し、共通化する
- **SOLID原則の適用**: 責務を分離し、設計を改善する
- **可読性の向上**: 変数名やメソッド名を改善する

**注意**: 一度に完璧にする必要はありません。段階的に改善していきます。

## TDDのサイクル(Red-Green-Refactor)

```
失敗するテストを書く(Red)
    ↓
テストを通す最小限の実装(Green)
    ↓
現在のテストがGreenになったか？
    ↓ Yes
既存のテストもGreenのままか？（リグレッションチェック）
    ↓ Yes
Refactorが必要か判断（チェックリスト確認）
    ↓ 必要
リファクタリング(Refactor)
    ↓
次のテストケースへ(Red)
    ↑
    | 不要
    └─────┘
```

**重要**: 1つのテストケースがGreenになるたびに、Refactorの必要性を判断します。既存のテストがすべてGreenのまま（リグレッションがない）であることを確認してから、Refactorステップに進みます。

## TDDの手順

### ステップ0: 最初のテストをどこから書くか決める
アーキテクチャパターンに応じて、最初のテストを書く場所を決定します。

**詳細ガイド**: [TDD実装の開始点ガイド](tdd-starting-points.md)

**簡単な判断基準**:
- **REST API / GraphQL**: Controller/Resolverから（Outside-In）
- **サーバーレス**: Serviceから（ビジネスロジック優先）
- **バッチ処理**: ドメインロジックから（Inside-Out）
- **イベント駆動**: イベントハンドラから（Outside-In）
- **CLI**: コアロジックから（Inside-Out）

### ステップ1: 要件を理解する
実装したい機能を明確に理解し、小さな単位に分解します。

### ステップ2: テストケースをリストアップ
実装すべきテストケースをリスト化します。簡単なものから順に実装していきます。

**重要**: 最もシンプルな正常系から始めること。

### ステップ3: 1つのテストを書く
最も簡単なテストケースから始めます。

### ステップ4: テストを実行して失敗を確認(Red)
テストが期待通りに失敗することを確認します。

### ステップ5: 最小限のコードを実装(Green)
テストを通過させるための最小限のコードを書きます。

### ステップ6: テストを実行して成功を確認
すべてのテストが通ることを確認します。

### ステップ7: Refactorが必要か判断する

**前提条件**:
- ✅ **現在のテストケース（1つ）がGreenになっている**
  - 1つのテストケース = 1つの振る舞いをテストするtest関数1つ
  - 例: `test('ユーザーを作成できる', () => { ... })`
- ✅ **既存のテストもすべてGreenのまま（リグレッションがない）**

**Refactorのタイミング:**
1つのテストケースがGreenになるたびに、Refactor判断チェックリストを確認します。
ただし、チェックリストに該当しない場合はRefactorをスキップして次のテストケースに進んでも構いません。

上記を満たした後、以下のチェックリストでRefactorの必要性を判断します。

#### Refactor判断チェックリスト

以下のいずれかに該当する場合、Refactorステップに進みます:

- [ ] **重複コードが3回以上出現している**（3回ルール）
  - 同じロジックが3箇所以上にある場合は共通化を検討
  - 2回程度なら様子見でもOK

- [ ] **マジックナンバー/文字列がある**
  - ハードコードされた数値や文字列を定数化すべき

- [ ] **関数が長すぎる（目安: 20行以上）**
  - 複数の責務を持っている可能性がある

- [ ] **ネストが深すぎる（3階層以上）**
  - 早期リターンや関数分割を検討

- [ ] **メソッドチェーンが長い（ドットが3つ以上連続）**
  - デメテルの法則違反の可能性

- [ ] **クラス/関数の責務が曖昧**
  - 単一責任の原則に違反している可能性

- [ ] **テストコードに重複がある**
  - ヘルパー関数やfixtureの抽出を検討

**重要**: 上記のいずれにも該当しない場合、Refactorをスキップして次のテストケースに進んでもOKです。

#### Refactorをスキップしても良い条件

以下の場合は、無理にRefactorする必要はありません:

- コードの重複が2回以下
- 関数が十分シンプル（10行以内）
- 変数名やメソッド名が明確で理解しやすい
- 次のテストケースで設計が変わる可能性が高い

**原則**: 問題がなければRefactorしない。Refactorは必要になってから行う（YAGNI）。

### ステップ8: リファクタリング(Refactor)

チェックリストで問題が見つかった場合のみ、コードを改善します。

**この段階で行うこと**:
- **DRY原則の適用**: 重複コードを除去し、共通化する
- **SOLID原則の適用**: 責務を分離し、設計を改善する
- **KISS原則の維持**: シンプルさを保つ（過度な抽象化を避ける）
- **デメテルの法則の適用**: メソッドチェーンを避け、カプセル化を保つ
- **可読性の向上**: 変数名やメソッド名を改善する

**注意**: 一度に完璧にする必要はありません。段階的に改善していきます。

### ステップ9: 繰り返し
次のテストケースに進み、ステップ3から繰り返します。

## サンプルコード例

### 例1: 計算機クラスの実装(JavaScript/TypeScript)

#### テストケースのリストアップ
- [ ] 2つの数値を足し算できる
- [ ] 2つの数値を引き算できる
- [ ] 2つの数値を掛け算できる
- [ ] 2つの数値を割り算できる
- [ ] 0で割った場合はエラーを投げる

#### Red: 失敗するテストを書く

```typescript
// calculator.test.ts
import { Calculator } from './calculator';

describe('Calculator', () => {
  let calculator: Calculator;

  beforeEach(() => {
    calculator = new Calculator();
  });

  test('2つの数値を足し算できる', () => {
    const result = calculator.add(2, 3);
    expect(result).toBe(5);
  });
});
```

この時点では`Calculator`クラスが存在しないため、テストは失敗します。

#### Green: テストを通す最小限の実装

```typescript
// calculator.ts
export class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
}
```

テストが通る最小限のコードを実装します。

#### 次のテストを追加(Red)

```typescript
// calculator.test.ts
test('2つの数値を引き算できる', () => {
  const result = calculator.subtract(5, 3);
  expect(result).toBe(2);
});
```

#### Green: 実装を追加

```typescript
// calculator.ts
export class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }

  subtract(a: number, b: number): number {
    return a - b;
  }
}
```

#### さらにテストを追加して実装を進める

```typescript
// calculator.test.ts
test('2つの数値を掛け算できる', () => {
  const result = calculator.multiply(2, 3);
  expect(result).toBe(6);
});

test('2つの数値を割り算できる', () => {
  const result = calculator.divide(6, 2);
  expect(result).toBe(3);
});

test('0で割った場合はエラーを投げる', () => {
  expect(() => calculator.divide(6, 0)).toThrow('0で割ることはできません');
});
```

```typescript
// calculator.ts
export class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }

  subtract(a: number, b: number): number {
    return a - b;
  }

  multiply(a: number, b: number): number {
    return a * b;
  }

  divide(a: number, b: number): number {
    if (b === 0) {
      throw new Error('0で割ることはできません');
    }
    return a / b;
  }
}
```

### 例2: ユーザー認証の実装(Python)

#### テストケースのリストアップ
- [ ] 正しいパスワードで認証成功
- [ ] 間違ったパスワードで認証失敗
- [ ] 空のパスワードで認証失敗
- [ ] 存在しないユーザーで認証失敗

#### Red: 失敗するテストを書く

```python
# test_authenticator.py
import unittest
from authenticator import Authenticator

class TestAuthenticator(unittest.TestCase):
    def setUp(self):
        self.auth = Authenticator()
        self.auth.register_user('user1', 'password123')

    def test_正しいパスワードで認証成功(self):
        result = self.auth.authenticate('user1', 'password123')
        self.assertTrue(result)
```

#### Green: テストを通す最小限の実装

```python
# authenticator.py
class Authenticator:
    def __init__(self):
        self.users = {}

    def register_user(self, username: str, password: str):
        self.users[username] = password

    def authenticate(self, username: str, password: str) -> bool:
        if username in self.users and self.users[username] == password:
            return True
        return False
```

#### 追加のテストと実装

```python
# test_authenticator.py
def test_間違ったパスワードで認証失敗(self):
    result = self.auth.authenticate('user1', 'wrongpassword')
    self.assertFalse(result)

def test_空のパスワードで認証失敗(self):
    result = self.auth.authenticate('user1', '')
    self.assertFalse(result)

def test_存在しないユーザーで認証失敗(self):
    result = self.auth.authenticate('nonexistent', 'password123')
    self.assertFalse(result)
```

すでに実装したコードでこれらのテストは通ります。

#### Refactor: セキュリティ向上のためのリファクタリング

```python
# authenticator.py
import hashlib

class Authenticator:
    def __init__(self):
        self.users = {}

    def _hash_password(self, password: str) -> str:
        """パスワードをハッシュ化"""
        return hashlib.sha256(password.encode()).hexdigest()

    def register_user(self, username: str, password: str):
        """ユーザーを登録(パスワードはハッシュ化して保存)"""
        if not username or not password:
            raise ValueError('ユーザー名とパスワードは必須です')
        self.users[username] = self._hash_password(password)

    def authenticate(self, username: str, password: str) -> bool:
        """ユーザー認証"""
        if not username or not password:
            return False
        if username not in self.users:
            return False
        return self.users[username] == self._hash_password(password)
```

テストも更新が必要です:

```python
# test_authenticator.py
def test_空のユーザー名で登録失敗(self):
    with self.assertRaises(ValueError):
        self.auth.register_user('', 'password123')

def test_空のパスワードで登録失敗(self):
    with self.assertRaises(ValueError):
        self.auth.register_user('user2', '')
```

## TDDのベストプラクティス

### 1. 小さなステップで進める
一度に大きな機能を実装しようとせず、小さな単位で進めます。

### 2. テストファーストを徹底する
必ずテストを先に書きます。実装を先に書いてしまうと、TDDのメリットが失われます。

### 3. 1つのテストで1つのことをテストする
テストは小さく、1つの振る舞いに焦点を当てます。

### 4. テストコードも綺麗に保つ
テストコードは実装コードと同じくらい重要です。読みやすく保守しやすいテストを書きます。

### 5. テストの独立性を保つ
各テストは他のテストに依存せず、どの順番で実行しても同じ結果になるようにします。

### 6. AAA(Arrange-Act-Assert)パターンを使う

```typescript
test('ユーザーを作成できる', () => {
  // Arrange: テストの準備
  const userService = new UserService();
  const userData = { name: 'John', email: 'john@example.com' };

  // Act: 実行
  const user = userService.createUser(userData);

  // Assert: 検証
  expect(user.name).toBe('John');
  expect(user.email).toBe('john@example.com');
});
```

### 7. テストの命名規則を守る
テスト名は「何をテストしているか」が明確にわかるようにします。

**良い例:**
```typescript
test('空の配列の場合は0を返す')
test('負の数を入力した場合はエラーを投げる')
test('有効なメールアドレスの場合はtrueを返す')
```

**悪い例:**
```typescript
test('test1')
test('check')
test('validation')
```

### 8. モックとスタブを適切に使う

```typescript
// モックの使用例
test('APIからユーザーデータを取得できる', async () => {
  // Arrange
  const mockApi = {
    fetchUser: jest.fn().mockResolvedValue({ id: 1, name: 'John' })
  };
  const userService = new UserService(mockApi);

  // Act
  const user = await userService.getUser(1);

  // Assert
  expect(user.name).toBe('John');
  expect(mockApi.fetchUser).toHaveBeenCalledWith(1);
});
```

### 9. エッジケースをテストする
正常系だけでなく、異常系やエッジケースもテストします。

```typescript
describe('divide', () => {
  test('正の数同士の割り算', () => {
    expect(divide(10, 2)).toBe(5);
  });

  test('負の数を含む割り算', () => {
    expect(divide(-10, 2)).toBe(-5);
  });

  test('0で割る場合', () => {
    expect(() => divide(10, 0)).toThrow();
  });

  test('小数点の結果', () => {
    expect(divide(10, 3)).toBeCloseTo(3.33, 2);
  });
});
```

### 10. テストカバレッジを意識する
コードカバレッジ100%が目標ではありませんが、重要なロジックは必ずテストでカバーします。

## TDDの利点

### 1. バグの早期発見
テストを先に書くことで、仕様の誤解や実装ミスを早期に発見できます。

### 2. リファクタリングの安全性
テストがあることで、安心してリファクタリングできます。

### 3. ドキュメントとしての役割
テストコードは、コードの使い方や仕様を示すドキュメントになります。

### 4. 設計の改善
テストしやすいコードは、疎結合で保守しやすいコードになります。

### 5. デバッグ時間の削減
問題が発生した箇所を素早く特定できます。

### 6. 自信を持ってコードを変更できる
テストがあることで、変更による影響範囲を把握しやすくなります。

## TDDの注意点

### 1. 初期コストが高い
テストを書く時間が必要なため、最初は遅く感じることがあります。

### 2. すべてに適用できるわけではない
UI、データベース、外部APIなど、テストが難しい領域もあります。

### 3. テストの保守コストがかかる
仕様変更時にはテストも更新する必要があります。

### 4. テストの品質が重要
悪いテストは、かえって開発の足を引っ張ることがあります。

## まとめ

TDDは以下のサイクルで進めます:

1. **Red**: 失敗するテストを書く
2. **Green**: テストを通す最小限のコードを書く
3. **Refactor**: コードを改善する

小さなステップで進め、テストファーストを徹底することで、品質の高いコードを効率的に開発できます。
