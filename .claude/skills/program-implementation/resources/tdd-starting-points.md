# TDD実装の開始点ガイド

## 概要

TDD（テスト駆動開発）を始める際、**どこから最初のテストを書くか**はアーキテクチャパターンによって異なります。このガイドでは、主要なアーキテクチャパターンごとに推奨される開始点を示します。

---

## 基本的なアプローチ

### Outside-In（外側から内側へ）
- **開始点**: ユーザーインターフェース層（Controller、Handler、Resolverなど）
- **進行方向**: UI → ビジネスロジック → データアクセス
- **メリット**: 実際に必要な機能だけを実装（YAGNI原則）、API仕様が先に固まる
- **デメリット**: 最初はモックが多くなる

### Inside-Out（内側から外側へ）
- **開始点**: ドメインロジック層（Service、VO、Entityなど）
- **進行方向**: ビジネスロジック → データアクセス → UI
- **メリット**: ビジネスルールを先に固められる、モックが少ない
- **デメリット**: 使われない機能を作るリスク（YAGNI違反）

---

## アーキテクチャパターン別ガイド

### 1. REST API（レイヤードアーキテクチャ）

**推奨アプローチ**: Outside-In

**実装順序**:
```
1. DTO/リクエスト・レスポンスの型定義（テストではない）
2. Controller（エンドポイント）のテスト
3. Service（ビジネスロジック）のテスト
4. Repository（データアクセス）のテスト
5. VO/Entityのバリデーション（必要に応じて）
```

**最初のテストの例**:
```typescript
// user.controller.test.ts
describe('UserController', () => {
  test('POST /users - ユーザーを作成できる', async () => {
    const mockService = {
      createUser: jest.fn().mockResolvedValue({
        id: '1',
        name: 'Test User',
        email: 'test@example.com'
      })
    };

    const controller = new UserController(mockService);
    const request = {
      name: 'Test User',
      email: 'test@example.com',
      password: 'password123'
    };

    const response = await controller.create(request);

    expect(response.id).toBe('1');
    expect(response.name).toBe('Test User');
  });
});
```

**理由**:
- エンドポイント仕様が明確
- フロントエンドとの契約が先に決まる
- 実際に必要なビジネスロジックだけを実装できる

---

### 2. サーバーレス（AWS Lambda / Cloud Functions）

**推奨アプローチ**: Outside-In（ただしHandlerは薄く保つ）

**実装順序**:
```
1. イベント型定義（API Gateway、S3、SQSなど）
2. Service（ビジネスロジック）のテスト ← ここから開始を推奨
3. 外部サービス連携（AWS SDK、外部API）のテスト
4. Handler（イベントハンドラ）のテスト
```

**最初のテストの例**:
```typescript
// image-processor.service.test.ts
describe('ImageProcessorService', () => {
  test('画像をリサイズできる', async () => {
    const mockS3 = {
      getObject: jest.fn().mockResolvedValue({ Body: Buffer.from('...') }),
      putObject: jest.fn().mockResolvedValue({})
    };

    const service = new ImageProcessorService(mockS3);
    const result = await service.resizeImage('bucket', 'key.jpg', 800, 600);

    expect(result.width).toBe(800);
    expect(result.height).toBe(600);
    expect(mockS3.putObject).toHaveBeenCalled();
  });
});
```

**理由**:
- Handlerは薄いグルーコード（イベント→Serviceへの変換のみ）
- ビジネスロジック（Service）が最も重要
- AWS SDKや外部サービスのモックが多いため、Serviceから始めると効率的

**Handlerのテスト例**（Serviceのテスト後）:
```typescript
// handler.test.ts
describe('S3ImageResizeHandler', () => {
  test('S3イベントを受け取って画像処理を実行', async () => {
    const mockService = {
      resizeImage: jest.fn().mockResolvedValue({ width: 800, height: 600 })
    };

    const event = {
      Records: [{
        s3: {
          bucket: { name: 'my-bucket' },
          object: { key: 'uploads/image.jpg' }
        }
      }]
    };

    const handler = new Handler(mockService);
    await handler.handle(event);

    expect(mockService.resizeImage).toHaveBeenCalledWith(
      'my-bucket',
      'uploads/image.jpg',
      800,
      600
    );
  });
});
```

---

### 3. バッチ処理（定期実行ジョブ）

**推奨アプローチ**: Inside-Out

**実装順序**:
```
1. ドメインロジック（Service/VO）のテスト ← ここから開始
2. データアクセス（Repository）のテスト
3. データ変換・集計ロジックのテスト
4. バッチジョブエントリーポイントのテスト
```

**最初のテストの例**:
```typescript
// sales-aggregator.service.test.ts
describe('SalesAggregatorService', () => {
  test('日次売上を集計できる', () => {
    const sales = [
      { amount: 1000, date: '2025-01-01' },
      { amount: 2000, date: '2025-01-01' },
      { amount: 1500, date: '2025-01-02' }
    ];

    const service = new SalesAggregatorService();
    const result = service.aggregateByDate(sales);

    expect(result['2025-01-01']).toBe(3000);
    expect(result['2025-01-02']).toBe(1500);
  });
});
```

**理由**:
- UIがないため、エンドポイントから始める意味がない
- データ処理の正確性が最優先
- ビジネスロジックが複雑なことが多い
- 純粋関数として実装しやすい（テストが書きやすい）

---

### 4. イベント駆動アーキテクチャ（Pub/Sub、メッセージキュー）

**推奨アプローチ**: Outside-In

**実装順序**:
```
1. イベントスキーマ定義（TypeScriptの型など）
2. イベントハンドラのテスト ← ここから開始
3. Service（ビジネスロジック）のテスト
4. Repository（データアクセス）のテスト
```

**最初のテストの例**:
```typescript
// order-created-handler.test.ts
describe('OrderCreatedHandler', () => {
  test('注文作成イベントを処理できる', async () => {
    const mockEmailService = {
      sendOrderConfirmation: jest.fn().mockResolvedValue(true)
    };

    const handler = new OrderCreatedHandler(mockEmailService);
    const event = {
      type: 'order.created',
      data: {
        orderId: '123',
        userId: '456',
        totalAmount: 5000
      }
    };

    await handler.handle(event);

    expect(mockEmailService.sendOrderConfirmation).toHaveBeenCalledWith({
      orderId: '123',
      userId: '456'
    });
  });
});
```

**理由**:
- イベントスキーマが契約（他のサービスとの境界）
- イベント駆動はメッセージ処理が中心
- ハンドラから始めることで、実際に必要な処理が明確になる

---

### 5. GraphQL API

**推奨アプローチ**: Outside-In

**実装順序**:
```
1. GraphQLスキーマ定義（schema.graphql）
2. Resolver（クエリ/ミューテーション）のテスト ← ここから開始
3. Service（ビジネスロジック）のテスト
4. Repository（データアクセス）のテスト
```

**最初のテストの例**:
```typescript
// user.resolver.test.ts
describe('UserResolver', () => {
  test('Query.user - ユーザーを取得できる', async () => {
    const mockService = {
      getUserById: jest.fn().mockResolvedValue({
        id: '1',
        name: 'Test User',
        email: 'test@example.com'
      })
    };

    const resolver = new UserResolver(mockService);
    const result = await resolver.user({ id: '1' });

    expect(result.id).toBe('1');
    expect(result.name).toBe('Test User');
    expect(mockService.getUserById).toHaveBeenCalledWith('1');
  });
});
```

**理由**:
- スキーマファースト設計が一般的
- Resolverがエントリーポイント
- REST APIと同様、Outside-Inが効果的

---

### 6. CLI/コマンドラインツール

**推奨アプローチ**: Inside-Out

**実装順序**:
```
1. コアロジック（Service/ドメインモデル）のテスト ← ここから開始
2. ファイルI/O、外部コマンド実行のテスト
3. CLIコマンドパーサーのテスト
4. エントリーポイント（main関数）のテスト
```

**最初のテストの例**:
```typescript
// markdown-converter.service.test.ts
describe('MarkdownConverterService', () => {
  test('MarkdownをHTMLに変換できる', () => {
    const service = new MarkdownConverterService();
    const markdown = '# Hello\n\nThis is **bold**.';
    const html = service.convert(markdown);

    expect(html).toContain('<h1>Hello</h1>');
    expect(html).toContain('<strong>bold</strong>');
  });
});
```

**理由**:
- コアロジックの再利用性が重要
- CLIは薄いラッパーに過ぎない
- ビジネスロジックを先に固めることで、他の用途（Web API化など）にも対応可能

---

## 選択フローチャート

```
アーキテクチャは？
│
├─ REST API / GraphQL
│   → Outside-In（Controller/Resolverから）
│
├─ サーバーレス（Lambda/Functions）
│   → Outside-In（ただしServiceから推奨）
│
├─ バッチ処理
│   → Inside-Out（ドメインロジックから）
│
├─ イベント駆動（Pub/Sub）
│   → Outside-In（イベントハンドラから）
│
└─ CLI/コマンドラインツール
    → Inside-Out（コアロジックから）
```

---

## 迷った時の判断基準

以下の質問で判断してください:

1. **明確なエントリーポイント（エンドポイント、イベント）があるか?**
   - Yes → Outside-In
   - No → Inside-Out

2. **ビジネスロジックが複雑か?**
   - Yes → Inside-Out（ロジックを先に固める）
   - No → Outside-In

3. **他のサービスやフロントエンドとの契約が先に決まっているか?**
   - Yes → Outside-In（契約から実装）
   - No → Inside-Out

4. **UIがあるか?**
   - Yes → Outside-In
   - No → Inside-Out

---

## 共通のベストプラクティス

どのアプローチを選んでも、以下は共通:

### 1. 最もシンプルなケースから始める
```typescript
// ✅ 良い例: 正常系の最もシンプルなケース
test('ユーザーを作成できる', () => { /* ... */ });

// ❌ 悪い例: いきなり複雑なケース
test('重複メールアドレスでバリデーションエラー、かつログ出力、かつメール送信失敗時のリトライ', () => {
  // 複雑すぎる!
});
```

### 2. テストケースをリストアップしてから実装
```markdown
## 実装するテストケース
- [ ] ユーザーを作成できる（正常系）
- [ ] 重複メールアドレスでエラーになる
- [ ] 無効なメールアドレスでエラーになる
- [ ] パスワードが短すぎるとエラーになる
```

### 3. 1テスト1検証
```typescript
// ✅ 良い例
test('ユーザーを作成できる', () => {
  const user = service.createUser({ name: 'Test', email: 'test@example.com' });
  expect(user.name).toBe('Test');
});

test('作成されたユーザーにはIDが付与される', () => {
  const user = service.createUser({ name: 'Test', email: 'test@example.com' });
  expect(user.id).toBeDefined();
});

// ❌ 悪い例: 1つのテストで複数の検証
test('ユーザー作成のすべてのチェック', () => {
  const user = service.createUser({ name: 'Test', email: 'test@example.com' });
  expect(user.name).toBe('Test');
  expect(user.id).toBeDefined();
  expect(user.createdAt).toBeInstanceOf(Date);
  expect(user.email).toBe('test@example.com');
  // 多すぎる!
});
```

### 4. テストの命名規則
```typescript
// パターン1: 「何ができるか」
test('ユーザーを作成できる', () => { /* ... */ });

// パターン2: 「条件 - 結果」
test('重複メールアドレス - バリデーションエラーを返す', () => { /* ... */ });

// パターン3: 「should + 動詞」（英語の場合）
test('should create a user', () => { /* ... */ });
```

---

## まとめ

### チェックリスト

実装開始前に以下を確認:

- [ ] アーキテクチャパターンを特定した
- [ ] Outside-In / Inside-Out どちらが適切か判断した
- [ ] 最初のテストケースを決めた（最もシンプルな正常系）
- [ ] テストケースのリストを作成した
- [ ] テスティングフレームワークを選定した（[参照](testing-frameworks.md)）

### 関連リソース

- [テスト駆動開発の原則](test-driven-development.md)
- [テスティングフレームワークの選定ガイド](testing-frameworks.md)
- [YAGNI原則](yagni.md)
