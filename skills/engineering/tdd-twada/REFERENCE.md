# t.wada流 TDD - Reference Guide

テストデザインパターン、コマンド、テンプレートの詳細リファレンス。

## Table of Contents

- [Test Design Patterns](#test-design-patterns)
- [Test Structure Template](#test-structure-template)
- [Commands](#commands)
- [pytest Configuration](#pytest-configuration)

---

## Test Design Patterns

### 境界値分析（Boundary Value Analysis）

境界値とその前後をテストすることで、エッジケースを見つける手法。

#### 基本パターン

```python
# 数値の境界
def test_最小値():
    assert validate_age(0) == True

def test_最小値マイナス1():
    assert validate_age(-1) == False

def test_最大値():
    assert validate_age(120) == True

def test_最大値プラス1():
    assert validate_age(121) == False
```

#### リストの境界

```python
def test_空のリスト():
    assert process([]) == []

def test_1要素のリスト():
    assert process([1]) == [1]

def test_2要素のリスト():
    assert process([1, 2]) == [1, 2]
```

#### 文字列の境界

```python
def test_空文字列():
    assert process("") == ""

def test_1文字():
    assert process("a") == "a"

def test_最大長():
    assert process("a" * 100) == "a" * 100

def test_最大長プラス1_で例外():
    with pytest.raises(ValueError):
        process("a" * 101)
```

### 等価クラス分割（Equivalence Partitioning）

入力空間を等価クラスに分割し、各クラスから代表値を選んでテスト。

```python
# 有効な年齢の代表値
def test_有効な年齢_子供():
    assert validate_age(10) == True  # 0-17の代表値

def test_有効な年齢_大人():
    assert validate_age(30) == True  # 18-120の代表値

# 無効な年齢の代表値
def test_無効な年齢_負の数():
    assert validate_age(-5) == False

def test_無効な年齢_超過():
    assert validate_age(150) == False
```

### 状態遷移テスト

状態を持つオブジェクトの遷移をテスト。

```python
def test_状態遷移_初期状態():
    order = Order()
    assert order.status == "pending"

def test_状態遷移_確認済み():
    order = Order()
    order.confirm()
    assert order.status == "confirmed"

def test_状態遷移_発送済み():
    order = Order()
    order.confirm()
    order.ship()
    assert order.status == "shipped"

def test_不正な遷移で例外():
    order = Order()
    with pytest.raises(InvalidStateTransition):
        order.ship()  # confirmせずにshipはできない
```

### 例外のテスト

```python
# 基本パターン
def test_不正な入力で例外():
    with pytest.raises(ValueError):
        parse_date("invalid")

# 例外メッセージの検証
def test_不正な入力で例外_メッセージ確認():
    with pytest.raises(ValueError) as exc_info:
        parse_date("invalid")
    assert "日付形式が不正" in str(exc_info.value)

# 特定の例外タイプ
def test_ファイルが存在しない():
    with pytest.raises(FileNotFoundError):
        read_file("nonexistent.txt")
```

---

## Test Structure Template

### 基本テンプレート

```python
"""
{モジュール名}のテスト

テスト対象: {クラス名/関数名}
"""
import pytest
from src.module import TargetClass


class TestTargetClass:
    """TargetClassの振る舞いテスト"""

    # ========== 正常系 ==========

    def test_基本的な使用方法(self):
        """最も一般的なユースケース"""
        # Arrange
        target = TargetClass()

        # Act
        result = target.do_something("input")

        # Assert
        assert result == "expected"

    def test_別のユースケース(self):
        """別の使用パターン"""
        target = TargetClass()
        result = target.do_something("other_input")
        assert result == "other_expected"

    # ========== 境界値 ==========

    def test_空入力(self):
        """空の入力を処理できる"""
        target = TargetClass()
        result = target.do_something("")
        assert result == ""

    def test_最大長入力(self):
        """最大長の入力を処理できる"""
        target = TargetClass()
        result = target.do_something("a" * 1000)
        assert len(result) == 1000

    # ========== 異常系 ==========

    def test_不正入力で例外(self):
        """不正な入力はValueErrorを発生"""
        target = TargetClass()
        with pytest.raises(ValueError):
            target.do_something(None)

    def test_型不一致で例外(self):
        """型が一致しない場合はTypeErrorを発生"""
        target = TargetClass()
        with pytest.raises(TypeError):
            target.do_something(123)  # 文字列を期待
```

### Fixture活用テンプレート

```python
import pytest
from src.module import User, UserRepository


@pytest.fixture
def user_repo():
    """テスト用リポジトリ"""
    repo = UserRepository(in_memory=True)
    yield repo
    repo.cleanup()


@pytest.fixture
def sample_user():
    """テスト用ユーザー"""
    return User(id="test-123", name="Test User", email="test@example.com")


class TestUserRepository:
    """UserRepositoryの振る舞いテスト"""

    def test_保存と取得(self, user_repo, sample_user):
        """ユーザーを保存して取得できる"""
        # Arrange
        # (fixtures で準備済み)

        # Act
        user_repo.save(sample_user)
        retrieved = user_repo.get(sample_user.id)

        # Assert
        assert retrieved == sample_user

    def test_存在しないユーザーはNone(self, user_repo):
        """存在しないユーザーはNoneを返す"""
        result = user_repo.get("nonexistent-id")
        assert result is None
```

### Parametrized Test Template

```python
import pytest


@pytest.mark.parametrize(
    "input_value,expected",
    [
        (0, 0),
        (1, 1),
        (2, 4),
        (3, 9),
        (-2, 4),
    ],
)
def test_平方計算(input_value, expected):
    """様々な入力で平方を計算"""
    result = square(input_value)
    assert result == expected


@pytest.mark.parametrize(
    "age,expected",
    [
        pytest.param(0, True, id="最小値"),
        pytest.param(50, True, id="中間値"),
        pytest.param(120, True, id="最大値"),
        pytest.param(-1, False, id="負の数"),
        pytest.param(121, False, id="超過"),
    ],
)
def test_年齢検証(age, expected):
    """年齢の妥当性を検証"""
    result = validate_age(age)
    assert result == expected
```

---

## Commands

### 基本的なテスト実行

```bash
# すべてのテストを実行
uv run pytest

# 特定ディレクトリのテストのみ
uv run pytest tests/unit

# 特定ファイルのテストのみ
uv run pytest tests/unit/test_user.py

# 特定テスト関数のみ
uv run pytest tests/unit/test_user.py::test_ユーザー作成

# 特定クラスのテストのみ
uv run pytest tests/unit/test_user.py::TestUser

# Verbose モード
uv run pytest -v

# 非常に詳細な出力
uv run pytest -vv
```

### テスト実行の制御

```bash
# 失敗したテストで停止
uv run pytest -x

# 最初の2つの失敗で停止
uv run pytest --maxfail=2

# 最後に失敗したテストから再実行
uv run pytest --lf

# 前回失敗したテストを先に実行
uv run pytest --ff

# キーワードでフィルタ
uv run pytest -k "test_user"

# マーカーでフィルタ
uv run pytest -m "slow"
```

### カバレッジ測定

```bash
# カバレッジ付きで実行
uv run pytest --cov=src

# カバレッジレポート（不足行を表示）
uv run pytest --cov=src --cov-report=term-missing

# HTML レポート生成
uv run pytest --cov=src --cov-report=html

# カバレッジ80%未満で失敗
uv run pytest --cov=src --cov-fail-under=80
```

### TDD向けWatch Mode

```bash
# ファイル変更時に自動実行
uv run ptw -- tests/unit -v

# 特定ディレクトリを監視
uv run ptw -- tests/unit --patterns="*.py"

# 失敗したテストのみ再実行
uv run ptw -- tests/unit --lf
```

### デバッグ

```bash
# 標準出力を表示（print文が見える）
uv run pytest -s

# pdbデバッガで停止
uv run pytest --pdb

# 最初の失敗でpdbに入る
uv run pytest -x --pdb

# トレースバックを長く表示
uv run pytest --tb=long
```

### その他の便利なオプション

```bash
# テストの実行時間を表示
uv run pytest --durations=10

# 並列実行（pytest-xdist必要）
uv run pytest -n auto

# ランダムな順序で実行（pytest-randomly必要）
uv run pytest --randomly-seed=12345

# テスト一覧を表示（実行しない）
uv run pytest --collect-only
```

---

## pytest Configuration

### pyproject.toml設定例

```toml
[tool.pytest.ini_options]
# テストディレクトリ
testpaths = ["tests"]

# テストファイルのパターン
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]

# デフォルトオプション
addopts = [
    "--verbose",              # 詳細表示
    "--strict-markers",       # 未定義マーカーでエラー
    "--tb=short",            # 短いトレースバック
    "--cov=src",             # カバレッジ測定
    "--cov-report=term-missing",  # 不足行を表示
]

# マーカー定義
markers = [
    "slow: 実行に時間がかかるテスト",
    "integration: 統合テスト",
    "unit: ユニットテスト",
]

# カバレッジ設定
[tool.coverage.run]
source = ["src"]
omit = [
    "*/tests/*",
    "*/__pycache__/*",
    "*/migrations/*",
]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
    "if __name__ == .__main__.:",
    "if TYPE_CHECKING:",
    "class .*\\bProtocol\\):",
    "@(abc\\.)?abstractmethod",
]

# カバレッジ目標
fail_under = 80
```

### カスタムマーカーの使用

```python
import pytest


@pytest.mark.slow
def test_時間がかかる処理():
    """このテストは実行に時間がかかる"""
    result = heavy_computation()
    assert result == expected


@pytest.mark.integration
def test_データベース統合():
    """データベースを使用する統合テスト"""
    db = connect_to_db()
    result = db.query("SELECT 1")
    assert result == 1
```

```bash
# slowマーカーを除外して実行
uv run pytest -m "not slow"

# integrationマーカーのみ実行
uv run pytest -m integration
```

---

## Fixture Best Practices

### Scope活用

```python
import pytest


@pytest.fixture(scope="function")  # デフォルト: 各テスト関数で実行
def temp_data():
    return {"key": "value"}


@pytest.fixture(scope="class")  # クラスごとに1回
def db_connection():
    conn = connect_to_db()
    yield conn
    conn.close()


@pytest.fixture(scope="module")  # モジュールごとに1回
def app_config():
    return load_config("test_config.yaml")


@pytest.fixture(scope="session")  # セッションごとに1回
def docker_services():
    start_docker()
    yield
    stop_docker()
```

### Autouse Fixture

```python
@pytest.fixture(autouse=True)
def reset_database():
    """各テスト前に自動的にDBをリセット"""
    db.reset()
    yield
    # テスト後の処理
```

### Fixture Chain

```python
@pytest.fixture
def user():
    return User(name="Test")


@pytest.fixture
def user_with_posts(user):
    """userフィクスチャを使用"""
    user.posts = [Post("First post")]
    return user


def test_ユーザーの投稿(user_with_posts):
    assert len(user_with_posts.posts) == 1
```

---

## Additional Resources

- [pytest公式ドキュメント](https://docs.pytest.org/)
- [pytest-cov](https://pytest-cov.readthedocs.io/)
- [pytest-watch](https://github.com/joeyespo/pytest-watch)
- [t.wadaのTDD講演資料](https://t-wada.hatenablog.jp/)
