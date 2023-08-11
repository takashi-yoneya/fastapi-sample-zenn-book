---
title: "Python(pytest)でテスト書くならfixture,conftest,paramerarizeを理解すると世界が一気に変わる" # 記事のタイトル
emoji: "🐍" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["python", "pytest"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# 概要
Pythonのテストライブラリといえばpytestが一般的です。
Python標準のuniitestとは異なり、クラスベースではなく関数ベースでテストコードを記述することが一般的ですが、fixture,conftest,paramerarizeを理解すると一気に世界が変わり、テスト体験が圧倒的に向上するため、これらの実装方法を紹介します。


# リポジトリ
本記事の説明に使用しているサンプルのテスト実装は、以下のリポジトリです。

https://github.com/takashi-yoneya/python-test-template

# 想定読者
PythonやGitの基本的な使い方を理解している方を想定しているため、基本的な用語説明は省略しています。

# 環境
エンジニアの利用率の高いmacOSを前提として説明していますので、その他の環境の方は随時読み替えてください。
開発環境はVSCODEの前提で説明しています。

# 使用するパッケージ
- pytest
- pytest_mock: mockを簡単に管理できるパッケージ

# fixture
テストを作る際に大きなウェイトを占めるのが、テストデータの準備等の事前作業です。
これらをテスト関数内で毎回定義することもできますが、テストの焦点がぶれてしまうため得策ではありません。
そこで、fixture(治具の意味)を使用することで、テスト関数に依存したテストデータやAPI連携用のHTTPクライアント,DBクライアントなどを簡単に定義することができます。

任意の関数にデコレーターとして以下のようにfixtureをセットすると、以下のように定義したfixtureをテストコードの関数名に指定することで、テストコードの前処理を定義することができます。

引数にfixtureの関数名を指定するのが、Pytest以外で馴染みが薄くわかりずらい点ですが、fixtureは戻り値を取ることもできるので、引数に指定することで、戻り値もテスト関数内で使用することができます。

以下の例では、test_func1を実行した場合に、any_func_name関数がまず実行された後に、test_func1の中身が実行されるようになります。

```python
# any_func_nameというfixtureが定義されます
@pytest.fixture
def any_func_name():
  ...
  return value


# テスト関数の引数にfixtureを指定して使用できます
def test_func1(any_func_name):
  value = any_func_value
```

一般的によく使われるパターンはテスト用のDBクライアントを作成する処理をfixtureで定義して、必要なテスト関数で呼び出す方法です。
以下の例では、test_func_with_db関数を実行すると、db関数(fixture)が前処理として呼び出されてテスト用のDBClientを定義できます。

```python
@pytest.fixture
def db():
  # 使用するDBクライアントの定義を行う
  db_client = ...
  return db_client

def test_func_with_db(db):
  # fixtureとして定義したdb関数はテスト関数の引数に指定して呼び出すことができる
  res = db.query.fetchall()
  assert len(res) == 5 
```

以下のようにautouse=Trueを指定した場合は、テスト関数にfixtureを指定せずとも、全てのテスト関数で自動的に実行されるため、常に実行したい初期処理などを定義する場合に使用します。
便利ではありますが、どのfixtureが実行されているか、テスト関数から辿るのが難しくなるため、注意して使用する必要があります。

```python
# autouse=Trueを付与すると、全てのテスト関数の前処理として自動的に実行されます。
@pytest.fixture(autouse=True)
def setup():
  ...


def test_func():
  ...

```

テストの後処理を行いたい場合は、以下のようにyieldを使用すると簡単です。
yieldはreturnとは違い、呼び出し元の処理が終わった後にまたyieldの場所に戻ってくることが出来る制御です。

```python
@pytest.fixture
def db():
  # 使用するDBクライアントの定義を行う
  db_client = ...
  # 任意の後処理を定義したい場合は、yieldを使うことで、テスト関数実行後に、yieldより下の行の処理を実行できます。
  yield db_client
  db_clinet.close()

```

# conftest.py
fixtureはテストモジュール内で定義しても良いですが、複数のモジュールを横断して使用したいような場合は、conftest.pyにfixtureを定義することで、複数のモジュールを横断してfixtureを使用することができます。

conftest.pyはimportでの読み込みは不要で、自動的に読み込まれます。
conftest.pyは階層構造で処理され「conftest.pyが存在する階層を含むそれ以下の階層にしか適用されない」という特性があります。

以下図のような構成を例とすると、test/(テストルート)直下に配置したconftest.py(a)はテスト全体に適用され、dir/直下に配置したconftest.py(b)はdir1/配下のみに適用されます。
test/di1/dir1-1/test_1-1.py のテストコードには、図内の(a),(b),(c)のconftest.pyが適用されます。
![conftest_tree](/images/conftest_tree.png)

上位階層のconftest.pyには、DBClientの定義が必須データの投入などのfixtureを記述しておき、各ディレクトリ直下のconftest.pyで必要なfixtureを定義していくと、fixtureの共通化と個別化が効果的に行えます。


# parameterise
1つのテスト関数で色々なパターンのデータでテストをしたい場合は多いですが、parameteriseを使用すると、簡単に実装できます。

ただし過度な共通化を行うと、わかりずらいテストになってしまう場合もあるので、注意が必要です。

以下の例では、１つのテスト関数で２種類のパタメータでテストを定義しています。
テスト関数に```pytest.mark.parametrize```デコーレータを記述し、第一引数に変数名、第二引数にパラメータを複数指定できます。
パラメータの指定は```pytest.param```を使用すると可読性があがります。```pytest.param```には```pytest.mark.parametrize```の第一引数で指定した変数名の順番で、テストで使用したいパラメータを指定できます。idを明示的に指定した場合は、```テストファイル名::テスト関数[id名]```のように実行できます。

```python
import pytest

@pytest.mark.parametrize(
    # ここで指定した変数名が、テスト関数内で使用可能
    [
        "data_in",
        "expected_status",
        "expected_data"
    ],
    # 複数のパラメーターを定義できる
    [
      # 上記の順番(data_in, expected_status, expected_data)で変数を記述する
      pytest.param(
        {"name": "test"},
        200,
        {"name": "test"},
        # idを明示的に指定した場合は、テストファイル名::テスト関数[id名]のように実行できる
        id="success"
      ),
      pytest.param(
        {},
        400,
        {"error": "nameは必須です"},
        id="fail_request_error"
      )
    ],
)
# pytest.mark.parametrizeの第一引数で指定した変数名は、かならずテストコードの引数に指定する必要がある。
def test_func1(data_in, expected_status, expected_data, authed_client):
    # action
    res = authed_client.post("/users/register", json=data_in)
    
    # assert
    assert res.status_code == expected_status
    assert res.json() == expected_data
```

<!-- # mocker
私は基本的にはMockは使いたくない派ですが、外部連携やテスト対応するのが非常に工数がかかる部分については
mockerを使ってMock化します。

pytestではpytest_mockをinstallすると簡単です。

Mockを多用してしまうと、実際の動作に関わらずMockが決まった値を返してしまい意味のないテストになってしまう場合があるので(偽陽性)、Mockするかの判断は慎重に行う必要があります。

Mock化には、patchを使用します。
mocker.patchに指定するPathが少し特殊ですが、テスト対象の関数のPath+Mockしたい関数をドットで繋ぐ形式で記述します。
patchの第一引数で指定した関数やメソッドを任意のMockに置き換えることができます。
また、patchしたObjectを保持しておくと、p.assert_once()のように、対象の処理が何度実行されたかなどについてもチェックすることができます。

```python
from pytest_mock import MockerFixture

def test_func1(mocker: MockerFixture):
  # patchを適用すると、指定した関数はreturn_valueに指定した値を返すようになる
  p = mocker.patch("app.module.external_api.get_wather", return_value=[{"hoge": "piyo"}])

  # mockした関数を実行する
  res = get_wather("test")

  # patchした関数が一度だけ実行されたことを確認する
  p.assert_once()

  # Mock化したので、patchのreturn_valueと値と一致する
  assert res == {"hoge": "piyo"}
```

```python
# app.module.external_api.py

def get_wather(code) -> dict[str, str]:
  ...
  return result


``` -->
