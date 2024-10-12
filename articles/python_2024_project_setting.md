---
title: "[2024年版]Pythonプロジェクト管理,linterをuvとRuffで超高速化" # 記事のタイトル
emoji: "🐍" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["python", "uv", "ruff", "linter"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
publication_name: where
---

# 概要

Python のプロジェクト管理は poetry, pyenv など複数のツールを組み合わせる必要があり煩雑な状況でしたが、uv の登場によりオールインワンで管理することが可能になりました。
また Linter、Formatter についても black, flake8, isort など複数のライブラリを組み合わせて使用することが一般的でしたが、すべてをオールインワン統合した Ruff の登場により、状況が一変し Ruff で全てを管理できるようになりました。
これらは Rust 製の高速なライブラリなため、従来と比較して高速に動作します。

本記事では、基本的な uv と Ruff の導入方法と、VSCode での設定方法を紹介します。

# リポジトリ

本記事の説明に使用しているサンプルの実装は、以下のリポジトリです。

TODO:

# 想定読者

Python や Git の基本的な使い方を理解している方を想定しているため、基本的な用語説明は省略しています。

# 環境

エンジニアの利用率の高い macOS を前提として説明していますので、その他の環境の方は随時読み替えてください。
開発環境は VSCode の前提で説明しています。

# 使用するパッケージ

- uv
- Ruff
- pre-commit

# uv をプロジェクトに導入する

uv を使用するためには、作業端末に uv をインストールする必要があります。

macOS の場合は以下のコマンドでインストールできます。

```bash
brew install uv
```

Linux の場合は以下のコマンドでインストールできます。(macOS も可能)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

オートコンプリート(ターミナルでの入力補完) を有効にするためには、使用するシェルに合わせて以下のコマンドのいずれかを実行します。

```bash
# bash
echo 'eval "$(uv generate-shell-completion bash)"' >> ~/.bashrc

# zsh
echo 'eval "$(uv generate-shell-completion zsh)"' >> ~/.zshrc

# fish
echo 'uv generate-shell-completion fish | source' >> ~/.config/fish/config.fish

# elvish
echo 'eval (uv generate-shell-completion elvish | slurp)' >> ~/.elvish/rc.elv
```

# uv のプロジェクト設定ファイル作成

Python では pyproject.toml を使用してプロジェクト全体の設定を管理します。pyproject.toml に uv の設定を記述することができます。
`uv init` を実行することで、プロジェクトの設定ファイルを作成することができます。

以下コマンドで Python のバージョンを指定することができます。

```bash
uv python pin {python_version}

# 例) 3.12 を指定する場合
uv python pin 3.12
```

poetry と同様に以下のコマンドでライブラリを追加することができます。

```bash
uv add {package_name}

# 開発時のみ使用するライブラリを追加する場合
uv add --dev {package_name}
```

以下サンプルは FastAPI プロジェクトの pyproject.toml の例です。
dependencies にはプロジェクトで使用するライブラリを記述します。dev-dependencies には開発時にのみ使用するライブラリを記述します。

pyproject.toml

```yml
[project]
name = "project name"
version = "0.1.0"
description = ""
readme = "README.md"
requires-python = ">=3.11"

# プロジェクトで使用するライブラリ
dependencies = [
  "pydantic >= 2.6.3, < 3",
  "pydantic-settings >= 2.2.1, < 3",
  "fastapi >= 0.110.0",
  "uvicorn >= 0.28.0",
]

# 開発時のみ使用するライブラリ
[tool.uv]
dev-dependencies = [
  "pytest >= 8",
  "pytest-mock >= 3.11.1",
  "mypy >= 1.5.1",
  "requests-mock >= 1.11.0",
]
```

# Ruff をプロジェクトに導入する

Ruff をプロジェクトにインストールする場合は、一般的に開発環境にのみインストールすることが多いため、`--dev` オプションを使用してインストールします。

```bash
uv add --dev ruff

```

# Ruff の設定ファイル

Ruff も同等に pyproject.toml に 設定を記述できます。設定例を以下の通りです。前述の uv の設定記述に続けて記述することができます。こちらの設定は FastAPI に設定されている ruff の設定を参考にしています。

select にはチェックするエラーの種類を指定します。"ALL"を指定するとすべてのエラーをチェックします。

まずは実行してみて、対処不要なエラーは ignore に追記していくなどして、プロジェクトに合わせて設定を調整していくと良いと思います。

pyporject.toml

```yml

[tool.ruff]
# 1行の最大文字数
line-length = 88

[tool.ruff.lint]
# チェックするエラーの種類
select = [
    "E",  # pycodestyle errors
    "W",  # pycodestyle warnings
    "F",  # pyflakes
    "I",  # isort
    "B",  # flake8-bugbear
    "C4",  # flake8-comprehensions
    "UP",  # pyupgrade
]
# 除外するエラーの種類
ignore = [
    "E501",  # line too long, handled by black
    "B008",  # do not perform function calls in argument defaults
    "C901",  # too complex
    "W191",  # indentation contains tabs
    "B904", # raise ... from ... になっていない場合のエラーを無視
]

# ファイルごとのエラー除外
[tool.ruff.lint.per-file-ignores]
"__init__.py" = ["F401"]

# isort の設定
[tool.ruff.lint.isort]
known-third-party = ["fastapi", "pydantic", "starlette"]

[tool.ruff.lint.pyupgrade]
# Python3.8互換のための設定
keep-runtime-typing = true
```

# VSCode での設定

Ruff では VSCode 用の以下の拡張機能が提供されており、保存時などに自動て format を適用したり、エラー箇所を強調表示することができます。

```text
名前: Ruff
ID: charliermarsh.ruff
説明: A Visual Studio Code extension with support for the Ruff linter.
バージョン: 2024.22.0
パブリッシャー: Astral Software
VS Marketplace リンク: https://marketplace.visualstudio.com/items?itemName=charliermarsh.ruff
```

以下のように VSCode の設定ファイルに記述することで、保存時に自動でフォーマットを適用することができます。

.vscode/settings.json

```json
{
  "[python]": {
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll": "explicit",
      "source.organizeImports": "explicit"
    },
    "editor.defaultFormatter": "charliermarsh.ruff"
  }
}
```

# Ruff を pre-commit で動作させる

pre-commit は commit 前にコード全体をチェックすることができるライブラリです。pre-commit で使用するライブラリは、pre-commit 固有の環境に install されるため、プロジェクトのライブラリ依存関係に影響を与えません。

pre-commit がインストールされていない場合は、以下のコマンドでインストールします。

```bash
brew install pre-commit
pre-commit install
```

pre-commit は.pre-commit-config.yaml(拡張子は必ず yaml。yml では NG) で設定を管理しています。ruff を追加するには、以下のように設定ファイルを作成もしくは追記します。

.pre-commit-config.yaml

```yml
repos:
  - repo: https://github.com/charliermarsh/ruff-pre-commit
    rev: v0.4.6
    hooks:
      - id: ruff
        args:
          - --fix
      - id: ruff-format
```

以下のコマンドで実行できます

```bash
pre-commit run
```

# まとめ

uv,Ruff を使用したプロジェクト管理方法を紹介しました。poetry, pyenv, Flake8, black, isort などのライブラリを組み合わせて使用する場合と比較して、２つのライブラリに集約でき、管理、設定が用意になり、かつ Rust 製のため高速に動作するため、今後の Python プロジェクトでの活用をご検討いただければと思います。
