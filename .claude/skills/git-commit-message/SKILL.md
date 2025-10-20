---
name: Git Commit Message Creator
description: Gitリポジトリの変更内容（git status、git diff）を分析し、Conventional Commits仕様に準拠した適切なコミットメッセージを生成する。ユーザーがコミットメッセージの作成・生成・提案を依頼したとき、コミット実行を依頼されたとき、または変更内容に基づいたメッセージが必要なときに使用する。
allowed-tools: Bash, Read, Grep, Glob, AskUserQuestion
---

# Git Commit Message Creator

## 概要

このスキルは、Gitリポジトリ内の変更されたファイルの内容を分析し、Conventional Commits仕様に準拠した適切なコミットメッセージを生成する。

**用語の定義**：
- **開発者**: このスキルを使用してコミットメッセージを生成する人（AIが対話する相手）
- **ユーザー**: ソフトウェアのエンドユーザー（開発者が作成するアプリケーションを使用する人）

### スキルの目的

1. 変更内容の自動分析とコミットタイプの適切な選択
2. Conventional Commits仕様に準拠したメッセージの生成
3. 一貫性のあるコミット履歴の構築支援
4. セマンティックバージョニングとの連携

### 使用タイミング

- ユーザーがコミットメッセージの生成を明示的に依頼したとき
- `git commit`前に適切なメッセージが必要なとき
- 既存のコミットメッセージのレビューや改善を求められたとき

## リソースファイル一覧

このスキルは、以下のリソースファイルで構成されています：

1. **[execution-phases.md](resources/execution-phases.md)** - 3つのフェーズの詳細な実行手順
2. **[conventional-commits-spec.md](resources/conventional-commits-spec.md)** - Conventional Commits仕様の完全な説明
3. **[type-selection-guide.md](resources/type-selection-guide.md)** - タイプ選択の詳細なガイドライン
4. **[edge-cases.md](resources/edge-cases.md)** - エッジケースと特殊な状況への対処法
5. **[examples.md](resources/examples.md)** - 実践的な例（5つの完全な例）

## スキルの実行手順

このスキルは、以下の3つのフェーズで動作します。詳細な実装手順は [execution-phases.md](resources/execution-phases.md) を参照してください。

### フェーズ1：情報収集

変更内容を取得し、分析します。

**主要なステップ**：
1. **変更内容の取得**: `git status`, `git diff`, `git log`を並列実行
2. **変更の分析**: 追加・削除・修正されたコードを分類
3. **セキュリティチェック**: 機密情報ファイルの早期検出

**詳細**: [execution-phases.md - フェーズ1：情報収集](resources/execution-phases.md#フェーズ1情報収集)

### フェーズ2：メッセージ作成

Conventional Commits形式のメッセージを作成します。

**主要なステップ**：
1. **コミットタイプの決定**: feat, fix, docs, etc.
2. **スコープの決定**: 変更の影響範囲（オプション）
3. **説明文の作成**: 簡潔な要約（日本語、30-40文字推奨）
4. **本文の作成**: 詳細な説明（必要な場合）

**詳細**: [execution-phases.md - フェーズ2：メッセージ作成](resources/execution-phases.md#フェーズ2メッセージ作成)

### フェーズ3：最終調整と提示

フッターの追加とコミット実行前の最終確認を行います。

**主要なステップ**：
1. **フッターの追加**: BREAKING CHANGE, Refs, Closes等
2. **複数変更の処理**: 関連性を判断し、必要に応じて分割
3. **セキュリティチェック（最終）**: ファイル内容の詳細検査
4. **メッセージの提示**: HEREDOCを使用したコミット実行

**詳細**: [execution-phases.md - フェーズ3：最終調整と提示](resources/execution-phases.md#フェーズ3最終調整と提示)

## Conventional Commits仕様（概要）

### コミットメッセージの構造

```text
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### 主要なコミットタイプ

- **feat**: 新機能の追加（MINOR）
- **fix**: バグ修正（PATCH）
- **docs**: ドキュメントのみの変更
- **style**: コードの意味に影響しない変更
- **refactor**: バグ修正や機能追加を伴わないコード改善
- **perf**: パフォーマンス改善
- **test**: テストの追加や修正
- **build**: ビルドシステムや外部依存関係の変更
- **ci**: CI設定の変更
- **chore**: その他の変更

**詳細な仕様**: [conventional-commits-spec.md](resources/conventional-commits-spec.md)

### セマンティックバージョニングとの関係

- **MAJOR** (X.0.0): 破壊的変更 ('BREAKING CHANGE:' または '!')
- **MINOR** (0.X.0): 新機能の追加 ('feat')
- **PATCH** (0.0.X): バグ修正 ('fix')

## タイプ選択ガイド

複数のタイプに該当する場合や判断に迷う場合：

### よくあるケース

- **バグ修正 + リファクタリング** → `fix` を優先
- **新機能 + ドキュメント** → `feat` を優先
- **複数の独立した変更** → コミットを分割
- **テストのみの変更** → `test` を使用
- **依存関係の更新**:
  - セキュリティ修正 → `fix(deps)`
  - 通常のアップデート → `chore(deps)`

**詳細なガイドライン**: [type-selection-guide.md](resources/type-selection-guide.md)

## エッジケースと特殊な状況

以下の特殊な状況に遭遇した場合は、該当するガイドラインを参照してください：

- **変更がない場合**: [edge-cases.md - 変更がない場合](resources/edge-cases.md#変更がない場合)
- **初回コミット**: [edge-cases.md - 初回コミット](resources/edge-cases.md#初回コミット)
- **セキュリティ問題**: [edge-cases.md - セキュリティ上問題のあるファイルが検出された場合](resources/edge-cases.md#セキュリティ上問題のあるファイルが検出された場合)
- **WIPコミット**: [edge-cases.md - WIP](resources/edge-cases.md#wipコミット)
- **マージコミット**: [edge-cases.md - マージコミット](resources/edge-cases.md#マージコミット)
- **巨大な変更**: [edge-cases.md - 巨大な変更の扱い](resources/edge-cases.md#巨大な変更の扱い)
- **複数著者**: [edge-cases.md - 複数の著者によるコミット](resources/edge-cases.md#複数の著者によるコミット)
- **revert**: [edge-cases.md - コミットの取り消し](resources/edge-cases.md#コミットの取り消しrevert)

**全エッジケース**: [edge-cases.md](resources/edge-cases.md)

## 実践例

実際の変更からコミットメッセージを生成する完全な例：

1. **例1：バグ修正** - シンプルなバグ修正の例
2. **例2：新機能** - 複数ファイルにまたがる機能追加
3. **例3：複数変更タイプ** - 独立した変更を分割する例
4. **例4：関連する変更** - 関連性のある変更をまとめる例
5. **例5：破壊的変更** - BREAKING CHANGEを含む例

**詳細な実践例**: [examples.md](resources/examples.md)

## クイックリファレンス

### コミットメッセージの基本ルール

1. **説明文の文末に句点（。）を付けない** - Conventional Commitsの標準慣習
   - **例外**: フェーズ1で`git log`を確認し、既存のコミットメッセージが句点を使用している場合はプロジェクトの慣習に従う
2. **日本語で記述**（技術用語は英語のまま）
3. **タイプは小文字** (`feat`, `fix`, etc.)
4. **BREAKING CHANGEは大文字** (`BREAKING CHANGE:`)
5. **破壊的変更には`!`を付ける** (`feat!:`, `fix!:`)

### HEREDOCを使用したコミット実行

```bash
git commit -m "$(cat <<'EOF'
feat(auth): ユーザー認証機能を追加

Google OAuth 2.0を使用した認証機能を追加した。

Refs: #123
EOF
)"
```

**重要**: `<<'EOF'`のシングルクォートは変数展開を防ぐために必須

### スコープの命名規則

- **既存の慣習を優先**: 過去のコミットから形式を確認
- **言語/フレームワークに応じて決定**:
  - JavaScript/TypeScript: ケバブケース (`user-service`)
  - Python/Ruby: スネークケース (`user_service`)
  - Java/C#: キャメルケース (`userService`)
  - 迷った場合: ケバブケース推奨

**詳細**: [execution-phases.md - スコープの決定](resources/execution-phases.md#5-スコープscopeの決定オプション)

### セキュリティチェックパターン

**ファイル名パターン**（フェーズ1）:
- `.env`, `credentials.json`, `secrets.yaml`
- `*.pem`, `*.key`, `id_rsa`
- `database.yml`, `db_config.*`

**コンテンツパターン**（フェーズ3）:
- キー名: `password=`, `token=`, `api_key=`, `SECRET_KEY`
- トークン: `sk-`, `ghp_`, `gho_`, `xoxb-`, `ya29.`
- AWS: `AKIA`で始まる20文字
- プライベートキー: `-----BEGIN.*PRIVATE KEY-----`

**詳細**: [execution-phases.md - セキュリティチェック](resources/execution-phases.md#3-セキュリティチェック早期警告)

### コミット前チェックリスト

AIがコミットメッセージを生成する際、以下の項目を確認すること：

- ✅ タイプは小文字（`feat`, not `Feat`）
- ✅ スコープは括弧で囲む（`feat(api):`, not `feat:api:`）
- ✅ コロンの後にスペース（`: description`, not `:description`）
- ✅ 説明文に句点なし（`機能を追加`, not `機能を追加。`）
  - 例外：既存のコミットメッセージで句点が使用されている場合は従う
- ✅ 破壊的変更には`!`（`feat!:`, not `feat:` with BREAKING CHANGE only）
- ✅ BREAKING CHANGEは大文字（not `Breaking Change:`）
- ✅ HEREDOCは`<<'EOF'`（シングルクォート付き）を使用

## 実行時の注意事項

### AIが実施すべきこと

1. **3つのフェーズを順番に実行**: 情報収集 → メッセージ作成 → 最終調整
2. **フェーズ1で並列実行を必ず使用**: 以下のコマンドを1つのメッセージ内で並列実行すること
   - `git status`
   - `git diff`
   - `git diff --staged`
   - `git log --oneline -10`

   **重要**: 逐次実行すると4回のラウンドトリップが必要になり、効率が悪い。並列実行により1回で完了する。
3. **適切なタイプを選択**: 変更内容を分析し、最も適切なタイプを決定
4. **セキュリティチェックを2段階で実施**: フェーズ1（ファイル名）とフェーズ3（内容）
5. **複数変更の関連性を判断**: 関連性がある場合は1つに、独立している場合は分割
6. **開発者の明示的な確認を得てからコミット実行**

### AIが実施してはならないこと

1. **WIPコミットの独自判断での作成**: 開発者が明示的に依頼した場合のみ
2. **セキュリティ警告を無視したコミット実行**: 開発者の明示的な承認が必要
3. **開発者の確認なしでのコミット実行**: 「コミットして」等の明示的な指示が必要
4. **Gitの履歴編集操作**: rebase, amend等は開発者自身で実施

### 複数コミット分割の判断

**関連性がある変更（1つのコミットにまとめる）**:
- バグ修正 + そのバグのテスト
- 新機能 + その機能のドキュメント
- リファクタリング + それに伴う型定義の更新

**独立した変更（複数コミットに分割）**:
- 異なる機能（ログイン機能 + 検索機能）
- 異なるバグ（パーサーのバグ + UIのバグ）
- 無関係なタイプ（API変更 + 無関係なタイポ修正）

**詳細**: [execution-phases.md - 複数の変更を含む場合の処理](resources/execution-phases.md#9-複数の変更を含む場合の処理)

## よくある質問

### Q: スコープは必須ですか？
A: いいえ、スコープはオプションです。変更の影響範囲が明確な場合に使用します。

### Q: 文末に句点（。）は必要ですか？
A: 説明文では**不要**です（Conventional Commitsの慣習）。本文とフッターでは**使用**します。

### Q: 英語と日本語のどちらで書くべきですか？
A: **日本語を推奨**しますが、技術用語（JSON、OAuth、API等）は英語のまま使用します。

### Q: 破壊的変更の`!`と`BREAKING CHANGE:`は両方必要ですか？
A: `!`のみでも有効ですが、詳細な説明が必要な場合は両方使用することを推奨します。

### Q: 複数の変更をどう扱うべきですか？
A: 関連性を判断します。関連性がある場合は1つにまとめ、独立している場合は分割します。詳細は [execution-phases.md - 複数の変更を含む場合の処理](resources/execution-phases.md#9-複数の変更を含む場合の処理) を参照。

### Q: HEREDOCを使わずにコミットメッセージを渡すとどうなりますか？
A: 複数行のメッセージが正しく解釈されず、変数展開や特殊文字の問題が発生する可能性があります。`<<'EOF'`のシングルクォートは特に重要で、これにより変数展開が防止され、メッセージ内の`$`、バッククォート、バックスラッシュ等が文字列としてそのまま扱われます。

### Q: スコープの括弧を忘れるとどうなりますか？
A: `feat:scope:` のようになり、無効なフォーマットになります。スコープは必ず括弧で囲む必要があります（例: `feat(scope):`）。スコープがない場合は括弧ごと省略します（例: `feat:`）。

## まとめ

このスキルを使用することで、以下が実現できます：

1. **一貫性のあるコミットメッセージ**: Conventional Commits仕様に準拠
2. **自動化ツールとの連携**: セマンティックバージョニングツールが正しく動作
3. **明確なコミット履歴**: 変更内容が明確に記録される
4. **セキュリティリスクの軽減**: 機密情報のコミットを防止
5. **効率的な開発フロー**: 適切なコミット分割とメッセージ生成

詳細な実装手順、仕様、ガイドライン、例については、各リソースファイルを参照してください。
