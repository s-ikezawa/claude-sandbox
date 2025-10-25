# テスティングフレームワークの選定ガイド

## 基本方針

### 1. プロジェクト固有の指定を最優先
- **CLAUDE.mdやREADME.mdに指定がある場合**: その指定に従う
- **package.jsonやrequirements.txtに既存の設定がある場合**: 既存のフレームワークを使用
- **プロジェクト内にテストファイルが存在する場合**: 既存のパターンに合わせる

### 2. 指定がない場合のデフォルト選択
プロジェクト固有の指定がない場合、以下のメジャーなフレームワークを使用します。

---

## 言語別デフォルトフレームワーク

### TypeScript / JavaScript

#### フレームワーク: **Vitest** (推奨) または **Jest**

**選定理由**:
- Vitest: 高速、ESM対応、設定が少ない
- Jest: 最も普及、豊富なエコシステム、成熟度が高い

**判断基準**:
- `vite.config.ts`または`vite.config.js`が存在する → Vitest
- `package.json`の`devDependencies`に`vite`がある → Vitest
- 上記以外 → Jest

**基本的な使い方**:

```typescript
// calculator.test.ts (Vitest/Jest共通)
import { describe, test, expect } from 'vitest'; // または 'jest'
import { Calculator } from './calculator';

describe('Calculator', () => {
  test('2つの数値を足し算できる', () => {
    const calculator = new Calculator();
    expect(calculator.add(2, 3)).toBe(5);
  });
});
```

**セットアップコマンド**:
```bash
# Vitest
npm install -D vitest

# Jest
npm install -D jest @types/jest ts-jest
```

---

### Python

#### フレームワーク: **pytest**

**選定理由**:
- 最も普及しているPythonテスティングフレームワーク
- シンプルな構文
- 豊富なプラグインエコシステム

**基本的な使い方**:

```python
# test_calculator.py
import pytest
from calculator import Calculator

class TestCalculator:
    def test_足し算ができる(self):
        calculator = Calculator()
        assert calculator.add(2, 3) == 5
```

**セットアップコマンド**:
```bash
pip install pytest
```

---

### Java

#### フレームワーク: **JUnit 5** (JUnit Jupiter)

**選定理由**:
- Java標準のテスティングフレームワーク
- 業界標準
- モダンなアノテーションとアサーション

**基本的な使い方**:

```java
// CalculatorTest.java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {
    @Test
    void 足し算ができる() {
        Calculator calculator = new Calculator();
        assertEquals(5, calculator.add(2, 3));
    }
}
```

**セットアップ** (Maven):
```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.10.0</version>
    <scope>test</scope>
</dependency>
```

---

### Go

#### フレームワーク: **testing** (標準ライブラリ)

**選定理由**:
- Go標準ライブラリ
- 追加インストール不要
- シンプルで軽量

**基本的な使い方**:

```go
// calculator_test.go
package calculator

import "testing"

func TestAdd(t *testing.T) {
    calculator := NewCalculator()
    result := calculator.Add(2, 3)
    expected := 5

    if result != expected {
        t.Errorf("Add(2, 3) = %d; want %d", result, expected)
    }
}
```

**実行コマンド**:
```bash
go test
```

---

### Rust

#### フレームワーク: **built-in test** (標準機能)

**選定理由**:
- Rustに組み込まれている
- 追加インストール不要
- cargoと統合されている

**基本的な使い方**:

```rust
// calculator.rs
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add() {
        let calculator = Calculator::new();
        assert_eq!(calculator.add(2, 3), 5);
    }
}
```

**実行コマンド**:
```bash
cargo test
```

---

### Ruby

#### フレームワーク: **RSpec**

**選定理由**:
- Ruby界で最も普及
- BDD (振る舞い駆動開発) スタイル
- 読みやすいテストコード

**基本的な使い方**:

```ruby
# calculator_spec.rb
require 'rspec'
require_relative 'calculator'

RSpec.describe Calculator do
  it '2つの数値を足し算できる' do
    calculator = Calculator.new
    expect(calculator.add(2, 3)).to eq(5)
  end
end
```

**セットアップコマンド**:
```bash
gem install rspec
```

---

### PHP

#### フレームワーク: **PHPUnit**

**選定理由**:
- PHP標準のテスティングフレームワーク
- Composer統合
- 豊富な機能

**基本的な使い方**:

```php
// CalculatorTest.php
use PHPUnit\Framework\TestCase;

class CalculatorTest extends TestCase
{
    public function test足し算ができる()
    {
        $calculator = new Calculator();
        $this->assertEquals(5, $calculator->add(2, 3));
    }
}
```

**セットアップコマンド**:
```bash
composer require --dev phpunit/phpunit
```

---

### C#

#### フレームワーク: **xUnit** または **NUnit**

**選定理由**:
- xUnit: モダンで.NET Coreに最適化
- NUnit: 成熟度が高く、豊富な機能

**判断基準**:
- .NET Core/5+ プロジェクト → xUnit
- .NET Framework → NUnit

**基本的な使い方** (xUnit):

```csharp
// CalculatorTests.cs
using Xunit;

public class CalculatorTests
{
    [Fact]
    public void 足し算ができる()
    {
        var calculator = new Calculator();
        Assert.Equal(5, calculator.Add(2, 3));
    }
}
```

**セットアップコマンド**:
```bash
dotnet add package xunit
dotnet add package xunit.runner.visualstudio
```

---

## プロジェクト固有設定の確認方法

### 1. CLAUDE.mdまたはREADME.mdを確認
```bash
# プロジェクトルートのドキュメントを確認
cat CLAUDE.md
cat README.md
```

### 2. 既存の設定ファイルを確認

**TypeScript/JavaScript**:
```bash
# package.json の scripts や devDependencies を確認
cat package.json | grep -E "(test|jest|vitest|mocha)"
```

**Python**:
```bash
# pytest設定ファイルを確認
cat pytest.ini
cat pyproject.toml | grep pytest
```

**Java**:
```bash
# pom.xml または build.gradle を確認
cat pom.xml | grep junit
```

### 3. 既存のテストファイルを確認
```bash
# テストファイルのパターンを確認
find . -name "*test*" -o -name "*spec*" | head -5
```

---

## TDD実装時の注意点

### テストファイルの命名規則

各フレームワークのデフォルト規則に従う:

| 言語/FW | テストファイル命名規則 |
|---------|----------------------|
| Vitest/Jest | `*.test.ts`, `*.spec.ts` |
| pytest | `test_*.py`, `*_test.py` |
| JUnit | `*Test.java` |
| Go testing | `*_test.go` |
| Rust | テストコードは同じファイル内の`#[cfg(test)]`モジュール |
| RSpec | `*_spec.rb` |
| PHPUnit | `*Test.php` |
| xUnit | `*Tests.cs` |

### テストの配置場所

**同階層配置 (推奨)**:
```
src/
  calculator.ts
  calculator.test.ts
```

**別ディレクトリ配置**:
```
src/
  calculator.ts
tests/
  calculator.test.ts
```

プロジェクトの既存パターンに合わせること。

---

## まとめ

### チェックリスト

実装開始前に以下を確認:

- [ ] CLAUDE.mdやREADME.mdにテストフレームワークの指定があるか?
- [ ] package.jsonやrequirements.txtに既存のテスト設定があるか?
- [ ] プロジェクト内に既存のテストファイルがあるか?
- [ ] 上記がない場合、言語に応じたデフォルトフレームワークを使用する

### 迷った時の判断基準

1. **既存のプロジェクト設定を最優先**
2. **チームの慣習を尊重**
3. **指定がない場合のみデフォルトを使用**
4. **迷ったらメジャーなフレームワークを選択** (上記リスト参照)
