---
title: "[2023年最新版]テストコードを制するものは、開発を制する(Python,Pytest)" # 記事のタイトル
emoji: "🐍" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["python"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# 概要
開発品質の良し悪しは、テスト体験によって決まると言っても過言ではありません。
テスト体験が良いと、積極的にテストコードが実装されますが、テスト体験が悪いと最低限のテストしか実装されなかったり、最悪はテスト自体実装されなくなってしまいます。

この記事では、私がブラッシュアップを続けてきた、PythonのPytestを使用したテストの書き方について紹介します。


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

# 基本
## pytestを使ったテストの構成、基本
pytestで良いテストを書けるかどうかは、fixtureとconftest.pyの理解度に左右されます。
Python標準のunittestと比較すると、少し特殊な構造になっているので、最初は戸惑いがちですが、慣れると非常に簡潔にテストが書けるようになっています。

## fixture
テストを作る際に大きなウェイトを占めるのが、テストデータの準備等の事前作業です。
これらをテスト関数内で毎回定義することもできますが、テストの焦点がぶれてしまうため得策ではありません。
そこで、fixture(治具の意味)を使用することで、テスト関数に依存したテストデータやAPI連携用のテストClient,DBクライアントなどを簡単に定義することができます。

任意の関数にデコレーターとして以下のようにfixtureをセットすると、以下のように定義したfixtureはテストコードなどで引数に関数名を指定して呼び出すことができます。

引数にfixtureの関数名を指定するのが、Pytest以外で馴染みが薄くわかりずらい点ですが、fixtureは戻り値を取ることもできるので、引数に指定することで、戻り値もテスト関数内で使用することができます。

```python
# any_func_nameというfixtureが定義されます
@pytest.fixture
def any_func_name():
  ...


# テスト関数の引数にfixtureを指定して使用できます
def test_func1(any_func_name):
  ...
```

一般的によく使われるパターンはテスト用のDBクライアントを作成する処理をfixtureで定義して、必要なテスト関数で呼び出す方法です。

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

以下のようにautouse=Trueを指定した場合は、テスト関数にfixtureを指定しなくとも、自動的に実行されます。
常に実行したい初期処理などを定義する場合に使用します。
便利ではあるが、どのfixtureが実行されているか、テスト関数から辿るのが難しくなるため、注意して使用する必要があります。

```python
@pytest.fixture(autouse=True)
def setup():
  ...


def test_func():
  ...

```


### conftest.py
fixtureはテストモジュール内で定義しても良いですが、複数のモジュールを横断して使用したいような場合は、conftest.pyにfixtureを定義することで、複数のモジュールを横断してfixtureを使用することができます。

conftest.pyは階層構造で処理されるので、conftest.pyが存在する階層以下にしか適用されない、という特性があります。
具体的には、fixtureの定義に使用することが多いです

test root -> 全体で使用するfixtureを定義
app1 -> app1のみで使用するfixtureを定義
DB連携を行う場合は、DBへのレコード投入を行うことが多いです。
これにより、テストするモジュール単位でDBレコードを使い分けることができます。


### parameterise
1つのロジックで色々なパターンのテストをしたい場合は多いですが、parameteriseを使用すると、簡単に実装できます。

ただし、共通化しすぎると、わかりずらいテストになってしまう場合もあるので、正常系と異常系を分けるなどの工夫は必要になると思います。

```python
import pytest

@pytest.mark.parametrize(
    [
        "data_in",
        "expected_status",
        "expected_data"
    ],
    [
      pytest.param(
        {"name": "test"},
        200,
        {"name": "test"},
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
def test_func1(data_in, expected_status, expected_data, authed_client):
    # action
    res = authed_client.post("/users/register", json=data_in)
    
    # assert
    assert res.status_code == expected_status
    assert res.json() == expected_data
```

### mocker
私は基本的にはMockは使いたくない派ですが、外部連携やテスト対応するのが非常に工数がかかる部分については
mockerを使ってMock化します。

Mockを不要に多用してしまうと、実際の動作に関わらずMockが決まった値を返してしまい意味のないテストになってしまう場合があるので、Mockするかの判断は慎重に行う必要があります。

mocker.patchに指定するPathが少し特殊ですが、Mockしたい関数がimportされているファイルからの想定Pathのような形で記述します。

```python
def test_func1(mocker: MockerFixture):
  p = mocker.patch("app.module.external_func", return_value=[])
  p.assert_once()
```

### ディレクトリ構成

# テストを作る流れ
## テストデータの準備(arrange)
テストデータの準備はconftest.pyで行います。Web開発においてはテストデータとはほぼ全てDBに投入されたデータを意味します。
そこで、conftest.pyでDBにデータを登録します。

```
```

## テストロジックの作成(action, assert)


## 1ロジックで複数パターンのテストを用意
parameterize


# testはarrangeが一番面倒
テストのAAAの原則でいうと
arange
action
assetion
のうち、arrangeが非常に面倒な場合が多いです
テストをするために必要なデータを準備して、テスト可能な状態にすることが必要ですが
これを失敗すると、テストでのみエラーが発生するといった本末転倒な自体に遭遇します。

そのため、前提の準備であるarrageを簡単に整備できるように方式を検討する必要があります。

共通化すべきは共通化するが過度な共通化は禁物
意図がテストコードがから明確に理解できるようにする


# 不用意にテストが壊れないこと
テストコードを書く開発を行なったことがあれば経験があると思いますが、実装と無関係なように見えるテストがなぜか落ちて
その対応に無意な時間を消費する、ということが多々発生します。

これは最悪にテスト体験を低下させるので、極力これが起きないような構成にする必要があります。

これの原因の多くはテストデータを過度に共有化していることに起因する場合が多いです。
今回のテスト用にテストデータを修正した際に、それを別のテストからも参照してたためにテストが落ちるというものです。

テストデータの共通化は最低限として、各テストコードのconftest.pyで個別のテストデータを定義するようにした方が、影響範囲を最小化できるのでオススメです。

修正しやすいこと