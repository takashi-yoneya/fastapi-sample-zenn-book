---
title: "[2023年最新版]Pythonを案件で使うなら、とりあえず入れるべきパッケージや構成" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["python"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# 概要
チームでPythonを開発する場合に活用可能な、パッケージや構成などの開発テンプレートを紹介します。
パッケージ管理、lint, test, loggingなどの、汎用的にプロジェクトで活用可能な構成になっています。

今回説明する内容のリポジトリは以下の通りです。

https://github.com/takashi-yoneya/python-template

# 想定読者

PythonやGitの基本的な使い方を理解している方を想定しているため、基本的な用語説明は省略しています。

# 環境
エンジニアの利用率の高いmacOSを前提として説明していますので、その他の環境の方は随時読み替えてください。
開発環境はVSCODEの前提で説明しています。

# 必須パッケージ
## pyenv

複数バージョンのPythonを管理することができる便利なツールです。
インストール方法は、以下の記事がわかりやすいです。
https://zenn.dev/kenghaya/articles/9f07914156fab5

## poetry

プロジェクト管理ができる便利なツールです。
node.jsでいうところのnpmのような位置づけです。

小規模の場合はpython標準のvenvでも問題はないですが、packageのlockファイルや、プロジェクト全体の設定管理ができるのでオススメです。

インストール方法は、以下の記事がわかりやすいです。
https://qiita.com/nokonoko_1203/items/a694be4e76da0872f51a#poetry%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%99%E3%82%8B


## pre-commit
commit時に自動で所定のチェックを行うことができるツールです。
必須ではないですが、Lintのチェック漏れを防止できるため、インストールを推奨します。

以下のコマンドでインストールできます。

```bash
brew install pre-commit
```

# 構成
## ディレクトリ構成 

ソースコードのディレクトリ構成は以下の通りです。

```
├── .env                    # 環境変数用ファイル（git管理外のため、.env.exampleからコピーする）
├── .env.example            # 環境変数用ファイルのサンプル
├── .env.prd                # 環境変数用ファイルの本番環境用
├── .pre-commit-config.yaml # pre-commit設定ファイル
├── LICENSE.md
├── Makefile                # タスクランナーの定義に使用
├── Readme.md
├── mypy.ini                # mypyの設定ファイル
├── poetry.lock             # poetyrのlockファイル
├── pyproject.toml          # project全体の設定ファイル
├── requirements-dev.txt    # poetryを使用しない場合のrequirements.txt(開発環境用)
├── requirements.txt        # poetryを使用しない場合のrequirements.txt
├── src                     # mainのソースコード
│   ├── common              # 全体で使用する処理をまとめたディレクトリ
│   │   ├── __init__.py
│   │   ├── configs.py      # ソースコード全体で使用する設定の管理
│   │   └── logger.py       # logger管理用
│   ├── logger_config.yaml  # logging設定ファイル
│   ├── main.py             # サンプルのmain
│   └── sub.py              # サンプルのsub関数
└── tests                   # テストコード格納用のディレクトリ
    ├── conftest.py
    └── sub
        ├── __init__.py
        ├── conftest.py
        └── test_sub.py
```

## プロジェクト管理
### pyenv,poetry
pyenvを使用すると複数のPythonバージョンを簡単に切り替えることができます。
複数のProjectを１つのPCで共存させたい場合に便利です。

python3.10をインストールし、poetryでpython3.10の仮装環境を作成するコマンドは以下の通りです。

```bash
	pyenv install 3.10
	poetry env use 3.10
```

既存のProjectからパッケージをインストールするコマンドは以下の通りです。
poetry.lockがある場合はlockファイルから、ない場合はpyproject.tomlの定義に従いインストールされます。
通常プロジェクトではpoetry.lockが共有される場合が殆どだと思います。

```bash
    poetry install
```

## 静的解析
静的解析パッケージを整備することで、バグを事前に発見できたり、見た目が整えられたりと、簡単にコードの品質を向上させることができます。

### mypy
型、構文などの静的チェックを行います。

以下のように、pre-commit-config.yamlに設定することで、commit時にチェックできます。

```yaml:pre-commit-config.yaml
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.2.0
    hooks:
      - id: mypy
        exclude: ^tests/|^any-path/ # mypyの除外ディレクトリ。正規表現で記述できる
        additional_dependencies: [pydantic, types-PyYAML==6.0.7]
```

### ruff

新しいPythonのLinter、Formatterです。

かなり多くの設定項目があるので、詳細は公式参照。

https://beta.ruff.rs/docs/settings/

注意点としてまだ一部のチェックには未対応なため、既存のパッケージと併用がオススメです。

設定はpyproject.tomlに記述可能です。設定の一例は以下の通りです。

```toml:pyproject.toml
[tool.ruff]
target-version = "py310 # python-version
src = ["src", "tests"] # ソースコードのあるディレクトリ
select = ["ALL"] # 有効かするチェックの種類。ALLの場合は全て有効
exclude = [".venv"] # チェックを除外するディレクトリ
ignore = [ # 無視するエラーコード 
    "G004", # `logging-f-string`
    "PLC1901", # compare-to-empty-string
    "PLR2004", # magic-value-comparison
    "ANN101", # missing-type-self
    "ANN102", # missing-type-cls
    "ANN002", # missing-type-args
    "ANN003", # missing-type-kwargs
    "ANN401", # any-type
    "ERA", # commented-out-code
    "ARG002", # unused-method-argument
    "INP001", # implicit-namespace-package
    "PGH004", # blanket-noqa
    "B008", # Dependsで使用するため
    "A002", # builtin-argument-shadowing
    "A003", # builtin-attribute-shadowing
    "PLR0913", # too-many-arguments
    "RSE", # flake8-raise
    "D", # pydocstyle
    "C90", # mccabe
    "T20", # flake8-print
    "SLF", #  flake8-self
    "BLE", # flake8-blind-except
    "FBT", # flake8-boolean-trap
    "TRY", # tryceratops
    "COM", # flake8-commas
    "S", # flake8-bandit
    "EM",#flake8-errmsg
    "EXE", # flake8-executable
    "ICN", # flake8-import-conventions
    "RET",#flake8-return
    "SIM",#flake8-simplify
    "TCH", # flake8-type-checking
    "PTH", #pathlibを使わないコードが多いので、除外する
    "ISC", #flake8-implicit-str-concat
    "N", # pep8-naming
    "PT", # flake8-pytest-style
]
line-length = 120 # １行の最大文字数
```

### black
従来からあるFormatterですが、ruffでは一部の機能が未実装なため、補完する目的でblackを使用します。

以下のように、pre-commit-config.yamlに設定することで、commit時にチェックできます。

```yaml:pre-commit-config.yaml
  - repo: https://github.com/psf/black
    rev: 23.3.0
    hooks:
      - id: black
        args: [src/]
```

## logging

簡単にloggingが使用できるように、configファイルを使用して、loggingを設定しています

以下の例では、normalというformatterを定義して、console用のHandlerにセットすることで、コンソールにログを出力されるようにしています。

```yaml:logger_config.yaml
version: 1
disable_existing_loggers: false

formatters:
  json:
    format: "%(asctime)s %(name)s %(levelname)s  %(message)s %(filename)s %(module)s %(funcName)s %(lineno)d"
    class: pythonjsonlogger.jsonlogger.JsonFormatter
  normal:
    format: "[%(asctime)s - %(levelname)s - %(filename)s(func:%(funcName)s, line:%(lineno)d)] %(message)s"

handlers:
  console:
    class: logging.StreamHandler
    level: INFO
    formatter: normal # json形式にしたい場合はjsonと記述する
    stream: ext://sys.stdout

loggers:
  src:
    level: INFO
    handlers: [console]
    propagate: false

root:
  level: INFO
  handlers: [console]

```

以下のように日時やログレベル、ファイル名、行数などの詳細情報が出力できるため、トラブル調査に役立ちます。

```
[2023-05-05 09:07:44,863 - INFO - main.py(func:main, line:24)] Start main
```

## pydantic
Pythonでも動的に型チェックが可能なpydanticです。Python標準のDataclassよりも汎用的に使用できるためオススメです。
FastAPIでは必須で使われるPydanticですが、FastAPI以外でも有用に活用することができます。

## test

pytestを使いparameterizeで１つのロジックで複数のテストケースを効率的に記述する方法を紹介します。

@pytest.mark.parametrizeのデコレーターを使用することで、複数のテストケースを１つのロジックで記述することができ、可読性、保守性が向上します。

```python
import pytest
from src.sub import sub_func


@pytest.mark.parametrize(
    ["text", "expected"],  # 変数名をリストで指定
    [
        pytest.param(
            "test1", "sub_func: test1", id="test1"  # 上で指定した順番で変数を指定  # テストケースの名前を指定
        ),
        pytest.param("test2", "sub_func: test2", id="test2"),
    ],
)
def test_sub_func(text: str, expected: str) -> None:
    """sub_funcのテスト
    parametrizeを使って、1つのロジックで複数のテストケースを定義できる
    """
    assert sub_func(text) == expected


```

## pre-commit
commit前にformatやlintを実行できます。
以下のように.pre-commit-config.yaml でcommit時に実行する処理を定義でき、pre-commitは独自の環境にパッケージをインストールするため、localのvenvにパッケージをインストールする必要はありません。

```yaml:.pre-commit-contig.yaml
# pre-commitの設定ファイル
repos:
  # ruffのチェック
  - repo: https://github.com/charliermarsh/ruff-pre-commit
    rev: "v0.0.264"
    hooks:
      - id: ruff
        args: [--fix, --exit-non-zero-on-fix]

  # ruffで未対応の部分をblackで補完する
  - repo: https://github.com/psf/black
    rev: 23.3.0
    hooks:
      - id: black
        args: [src/]

  # mypyのチェック
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.2.0
    hooks:
      - id: mypy
        exclude: ^tests/|^any-path/ # mypyの除外ディレクトリ。正規表現で記述できる
        additional_dependencies: [pydantic, types-PyYAML==6.0.7]
```

commit時にpre-commitを自動で実行するためには、以下のコマンドを１度実行してください。

```bash
pre-commit install
```

以降はcommit時にpre-commitが自動で実行されるようになり、失敗するとcommitできなくなります。
検証中などでpre-commitを無視してcommmitしたい場合は以下のように-nオプションを付与すると、pre-commitが無視されます。

```bash
git commit -n -m "comment"
```

## Makefile

Pythonの機能ではないですが、linuxやmacOSの場合はMakefileをタスクランナーとして使用できます。

.PHONYというダミーターゲットを使用することで、任意のコマンドを定義できます。

以下のinstallコマンドは、```make install```で実行できます。

installコマンドの定義

```makefile
# install
.PHONY: install
install:
	pyenv install 3.10
	poetry env use 3.10
	poetry install
	pre-commit install
```

# 実行方法

以下のリポジトリをクローン後、以下を実行して必要パッケージ等をインストールします。
https://github.com/takashi-yoneya/python-template

```bash
make install
```

以下のコマンドを実行して、コンソールでログが表示されたら成功です。

```bash
make run_main
```

表示されるログの例です。loggingを使用しています。

```
[2023-05-05 09:07:44,863 - INFO - main.py(func:main, line:24)] Start main
[2023-05-05 09:07:44,863 - INFO - main.py(func:main, line:25)] datetime.datetime.now(tz=datetime.timezone.utc)=datetime.datetime(2023, 5, 5, 0, 7, 44, 863376, tzinfo=datetime.timezone.utc)
[2023-05-05 09:07:44,863 - INFO - main.py(func:main, line:26)] os.path.join("src", "main.py")='src/main.py'
[2023-05-05 09:07:44,863 - INFO - sub.py(func:sub_func, line:7)] sub_func
[2023-05-05 09:07:44,863 - INFO - main.py(func:main, line:27)] sub_func(text="Hello World!")='sub_func: Hello World!'
[2023-05-05 09:07:44,863 - INFO - main.py(func:main, line:29)] id=1 name='Takashi Yoneya' job='Software Engineer'
[2023-05-05 09:07:44,863 - WARNING - main.py(func:main, line:30)] This is warning message.
[2023-05-05 09:07:44,863 - ERROR - main.py(func:main, line:31)] This is error message.
[2023-05-05 09:07:44,863 - ERROR - main.py(func:main, line:35)] division by zero
Traceback (most recent call last):
  File "/Users/yoneya/Projects/python-structure/src/main.py", line 33, in main
    1 / 0  # noqa
ZeroDivisionError: division by zero
[2023-05-05 09:07:44,863 - INFO - main.py(func:main, line:36)] End main
```