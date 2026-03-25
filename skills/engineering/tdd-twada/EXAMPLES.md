# t.wada流 TDD - Practical Examples

実用的なTDD例、AAA構造、アンチパターンの詳細。

## Table of Contents

- [Complete TDD Example](#complete-tdd-example)
- [AAA Pattern Examples](#aaa-pattern-examples)
- [Anti-Patterns](#anti-patterns)
- [Real-World Scenarios](#real-world-scenarios)

---

## Complete TDD Example

### シナリオ: 数値リストのソート関数

要件: 数値のリストを昇順でソートする関数を実装

#### Step 1: テストケースリスト作成

```
□ 空のリストを渡すと空のリストを返す
□ 1要素のリストはそのまま返す
□ 2要素の降順リストを昇順に並び替える
□ 複数要素をソートする
□ 重複要素を正しく処理する
□ 負の数を含んでもソートできる
```

#### Step 2: 🔴 Red - 最初のテスト

```python
# tests/unit/test_sort_numbers.py
def test_空のリストを渡すと空のリストを返す():
    # Arrange
    input_list = []

    # Act
    result = sort_numbers(input_list)

    # Assert
    assert result == []
```

**実行**: `pytest tests/unit/test_sort_numbers.py -v`
```
FAILED - NameError: name 'sort_numbers' is not defined
```
✅ 失敗を確認！

#### Step 3: 🟢 Green - 最小実装

```python
# src/sort.py
def sort_numbers(numbers):
    return []  # 仮実装: 常に空リストを返す
```

**実行**: `pytest tests/unit/test_sort_numbers.py -v`
```
PASSED
```
✅ テスト通過！

#### Step 4: 🔵 Refactor

このステップでは特に改善点なし。次のテストへ。

---

#### Step 5: 🔴 Red - 2つ目のテスト

```python
def test_1要素のリストはそのまま返す():
    # Arrange
    input_list = [42]

    # Act
    result = sort_numbers(input_list)

    # Assert
    assert result == [42]
```

**実行**: `pytest -v`
```
FAILED - assert [] == [42]
```
✅ 失敗を確認！

#### Step 6: 🟢 Green - 実装改善

```python
def sort_numbers(numbers):
    if len(numbers) == 0:
        return []
    return numbers  # 1要素の場合はそのまま返す
```

**実行**: `pytest -v`
```
PASSED (2 tests)
```
✅ すべてのテスト通過！

---

#### Step 7: 🔴 Red - 3つ目のテスト

```python
def test_2要素の降順リストを昇順に並び替える():
    # Arrange
    input_list = [2, 1]

    # Act
    result = sort_numbers(input_list)

    # Assert
    assert result == [1, 2]
```

**実行**: `pytest -v`
```
FAILED - assert [2, 1] == [1, 2]
```
✅ 失敗を確認！

#### Step 8: 🟢 Green - 本格実装

```python
def sort_numbers(numbers):
    return sorted(numbers)  # Pythonの組み込み関数を使用
```

**実行**: `pytest -v`
```
PASSED (3 tests)
```
✅ すべてのテスト通過！

---

#### Step 9: 🔵 Refactor

コードはシンプルで改善点なし。残りのテストを追加。

```python
def test_複数要素をソートする():
    # Arrange
    input_list = [5, 2, 8, 1, 9]

    # Act
    result = sort_numbers(input_list)

    # Assert
    assert result == [1, 2, 5, 8, 9]


def test_重複要素を正しく処理する():
    # Arrange
    input_list = [3, 1, 2, 1, 3]

    # Act
    result = sort_numbers(input_list)

    # Assert
    assert result == [1, 1, 2, 3, 3]


def test_負の数を含んでもソートできる():
    # Arrange
    input_list = [-5, 3, -1, 0, 2]

    # Act
    result = sort_numbers(input_list)

    # Assert
    assert result == [-5, -1, 0, 2, 3]
```

**実行**: `pytest -v`
```
PASSED (6 tests)
```
✅ すべてのテスト通過！TDD完了！

---

## AAA Pattern Examples

### Arrange-Act-Assert（AAA）構造

#### Good Example ✅

```python
def test_ユーザー作成():
    # Arrange（準備）: テストデータとオブジェクトの準備
    user_data = {"name": "Alice", "email": "alice@example.com"}
    user_service = UserService()

    # Act（実行）: テスト対象の操作を実行
    user = user_service.create_user(user_data)

    # Assert（検証）: 結果を検証
    assert user.name == "Alice"
    assert user.email == "alice@example.com"
    assert user.id is not None
```

**ポイント**:
- 各ブロックが明確に分離されている
- コメントでArrange-Act-Assertを明示
- 1つのActブロックのみ

#### Bad Example ❌

```python
def test_ユーザー作成():
    # AAA構造が不明瞭
    user_service = UserService()
    user = user_service.create_user({"name": "Alice", "email": "alice@example.com"})
    assert user.name == "Alice"
    another_user = user_service.create_user({"name": "Bob"})  # 複数のAct
    assert another_user.name == "Bob"
```

**問題点**:
- AAA構造が不明瞭
- 複数の振る舞いをテスト（Actが2回）
- テストが何を検証したいのか不明確

---

### AAA with Fixtures

```python
import pytest


@pytest.fixture
def user_service():
    """Arrange: テスト用サービスを準備"""
    return UserService(db=InMemoryDB())


@pytest.fixture
def valid_user_data():
    """Arrange: テスト用データを準備"""
    return {"name": "Alice", "email": "alice@example.com"}


def test_ユーザー作成(user_service, valid_user_data):
    # Arrange（fixtureで準備済み）
    # （追加の準備が必要な場合はここに記述）

    # Act
    user = user_service.create_user(valid_user_data)

    # Assert
    assert user.name == valid_user_data["name"]
    assert user.email == valid_user_data["email"]
```

---

## Anti-Patterns

### ❌ Anti-Pattern 1: 実装詳細をテストする

#### Bad Example

```python
class UserService:
    def __init__(self):
        self._cache = {}  # 内部キャッシュ

    def get_user(self, user_id):
        if user_id in self._cache:
            return self._cache[user_id]
        user = self._fetch_from_db(user_id)
        self._cache[user_id] = user
        return user


# ❌ 悪い例: 内部実装に依存
def test_キャッシュが更新される():
    service = UserService()
    user = service.get_user("123")
    assert "123" in service._cache  # 内部実装に依存！
```

**問題点**: 内部実装（`_cache`）に依存しているため、実装変更に脆弱。

#### Good Example ✅

```python
# ✅ 良い例: 振る舞いをテスト
def test_同じユーザーを2回取得しても同じ結果():
    # Arrange
    service = UserService()

    # Act
    user1 = service.get_user("123")
    user2 = service.get_user("123")

    # Assert
    assert user1 == user2  # 振る舞いのみテスト
```

---

### ❌ Anti-Pattern 2: テスト間の依存

#### Bad Example

```python
# ❌ 悪い例: テスト間で状態を共有
counter = 0


def test_カウンターを増やす():
    global counter
    counter += 1
    assert counter == 1


def test_カウンターを2回増やす():
    global counter
    counter += 1  # test_カウンターを増やす の後に実行される前提
    assert counter == 2  # 前のテストに依存！
```

**問題点**:
- テストの実行順序に依存
- 個別に実行できない
- デバッグが困難

#### Good Example ✅

```python
# ✅ 良い例: 各テストが独立
def test_カウンターを増やす():
    # Arrange
    counter = Counter()

    # Act
    counter.increment()

    # Assert
    assert counter.value == 1


def test_カウンターを2回増やす():
    # Arrange
    counter = Counter()

    # Act
    counter.increment()
    counter.increment()

    # Assert
    assert counter.value == 2
```

---

### ❌ Anti-Pattern 3: 過度なモック

#### Bad Example

```python
# ❌ 悪い例: 全てをモック化
def test_ユーザー登録処理():
    # すべてをモック化
    mock_validator = Mock()
    mock_hasher = Mock()
    mock_db = Mock()
    mock_email = Mock()
    mock_logger = Mock()

    # テストが実装の詳細に依存しすぎている
    service = UserService(
        validator=mock_validator,
        hasher=mock_hasher,
        db=mock_db,
        email=mock_email,
        logger=mock_logger
    )

    # ...複雑なモック設定が続く...
```

**問題点**: モックが多すぎてテストの意図が不明確。リファクタリングに脆弱。

#### Good Example ✅

```python
# ✅ 良い例: 外部依存のみモック化
def test_ユーザー登録処理():
    # Arrange
    mock_email_service = Mock()  # 外部サービスのみモック
    service = UserService(email_service=mock_email_service)
    user_data = {"name": "Alice", "email": "alice@example.com"}

    # Act
    user = service.register(user_data)

    # Assert
    assert user.name == "Alice"
    mock_email_service.send_welcome_email.assert_called_once()
```

---

### ❌ Anti-Pattern 4: 巨大なテスト

#### Bad Example

```python
# ❌ 悪い例: 1つのテストで複数の振る舞いを検証
def test_ユーザー管理():
    # 作成をテスト
    user = create_user("Alice")
    assert user.name == "Alice"

    # 更新をテスト
    update_user(user.id, {"name": "Bob"})
    updated_user = get_user(user.id)
    assert updated_user.name == "Bob"

    # 削除をテスト
    delete_user(user.id)
    assert get_user(user.id) is None
```

**問題点**:
- 複数の振る舞いをテスト
- どこで失敗したか分かりにくい
- テスト名から意図が不明

#### Good Example ✅

```python
# ✅ 良い例: 1テスト1振る舞い
def test_ユーザーを作成できる():
    # Arrange
    user_data = {"name": "Alice"}

    # Act
    user = create_user(user_data)

    # Assert
    assert user.name == "Alice"


def test_ユーザー名を更新できる():
    # Arrange
    user = create_user({"name": "Alice"})

    # Act
    update_user(user.id, {"name": "Bob"})
    updated_user = get_user(user.id)

    # Assert
    assert updated_user.name == "Bob"


def test_ユーザーを削除できる():
    # Arrange
    user = create_user({"name": "Alice"})

    # Act
    delete_user(user.id)

    # Assert
    assert get_user(user.id) is None
```

---

## Real-World Scenarios

### シナリオ1: 電卓クラスのTDD

```python
# ========== Test 1: 🔴 Red ==========
def test_足し算():
    # Arrange
    calc = Calculator()

    # Act
    result = calc.add(2, 3)

    # Assert
    assert result == 5


# ========== 🟢 Green ==========
class Calculator:
    def add(self, a, b):
        return a + b


# ========== Test 2: 🔴 Red ==========
def test_引き算():
    calc = Calculator()
    result = calc.subtract(5, 3)
    assert result == 2


# ========== 🟢 Green ==========
class Calculator:
    def add(self, a, b):
        return a + b

    def subtract(self, a, b):
        return a - b


# ========== Test 3: 🔴 Red ==========
def test_ゼロ除算で例外():
    calc = Calculator()
    with pytest.raises(ZeroDivisionError):
        calc.divide(5, 0)


# ========== 🟢 Green ==========
class Calculator:
    def add(self, a, b):
        return a + b

    def subtract(self, a, b):
        return a - b

    def divide(self, a, b):
        if b == 0:
            raise ZeroDivisionError("ゼロで除算できません")
        return a / b
```

---

### シナリオ2: バリデーションのTDD

```python
# ========== 要件 ==========
# メールアドレスのバリデーション関数
# - @ を含む
# - ドメインがある
# - 空文字はNG

# ========== Test 1: 🔴 Red ==========
def test_有効なメールアドレス():
    # Arrange
    email = "user@example.com"

    # Act
    result = validate_email(email)

    # Assert
    assert result is True


# ========== 🟢 Green ==========
def validate_email(email):
    return "@" in email and "." in email.split("@")[1]


# ========== Test 2: 🔴 Red ==========
def test_アットマークなしは無効():
    email = "userexample.com"
    result = validate_email(email)
    assert result is False


# (すでにパスする)

# ========== Test 3: 🔴 Red ==========
def test_ドメインなしは無効():
    email = "user@"
    result = validate_email(email)
    assert result is False


# ========== 🟢 Green（エラー処理追加） ==========
def validate_email(email):
    if "@" not in email:
        return False
    parts = email.split("@")
    if len(parts) != 2:
        return False
    domain = parts[1]
    return "." in domain


# ========== Test 4: 🔴 Red ==========
def test_空文字は無効():
    email = ""
    result = validate_email(email)
    assert result is False


# ========== 🟢 Green ==========
def validate_email(email):
    if not email or "@" not in email:
        return False
    parts = email.split("@")
    if len(parts) != 2:
        return False
    domain = parts[1]
    return "." in domain


# ========== 🔵 Refactor ==========
def validate_email(email):
    """メールアドレスの妥当性を検証

    Args:
        email: 検証するメールアドレス

    Returns:
        bool: 有効な場合True、無効な場合False
    """
    if not email:
        return False

    if "@" not in email:
        return False

    local, domain = email.split("@", 1)

    if not local or not domain:
        return False

    if "." not in domain:
        return False

    return True
```

---

### シナリオ3: 状態を持つオブジェクトのTDD

```python
# ========== 要件 ==========
# ショッピングカート
# - 商品を追加できる
# - 商品を削除できる
# - 合計金額を計算できる

# ========== Test 1: 🔴 Red ==========
def test_空のカートの合計は0():
    # Arrange
    cart = ShoppingCart()

    # Act
    total = cart.get_total()

    # Assert
    assert total == 0


# ========== 🟢 Green ==========
class ShoppingCart:
    def __init__(self):
        self.items = []

    def get_total(self):
        return 0


# ========== Test 2: 🔴 Red ==========
def test_商品を追加すると合計に反映される():
    # Arrange
    cart = ShoppingCart()

    # Act
    cart.add_item("Apple", 100)
    total = cart.get_total()

    # Assert
    assert total == 100


# ========== 🟢 Green ==========
class ShoppingCart:
    def __init__(self):
        self.items = []

    def add_item(self, name, price):
        self.items.append({"name": name, "price": price})

    def get_total(self):
        return sum(item["price"] for item in self.items)


# ========== Test 3: 🔴 Red ==========
def test_複数商品の合計():
    cart = ShoppingCart()
    cart.add_item("Apple", 100)
    cart.add_item("Banana", 50)
    assert cart.get_total() == 150


# (すでにパスする)

# ========== Test 4: 🔴 Red ==========
def test_商品を削除すると合計が減る():
    # Arrange
    cart = ShoppingCart()
    cart.add_item("Apple", 100)
    cart.add_item("Banana", 50)

    # Act
    cart.remove_item("Apple")
    total = cart.get_total()

    # Assert
    assert total == 50


# ========== 🟢 Green ==========
class ShoppingCart:
    def __init__(self):
        self.items = []

    def add_item(self, name, price):
        self.items.append({"name": name, "price": price})

    def remove_item(self, name):
        self.items = [item for item in self.items if item["name"] != name]

    def get_total(self):
        return sum(item["price"] for item in self.items)
```

---

This completes the practical examples! これらの例でt.wada流TDDの実践方法が理解できるはずだよぅ。
