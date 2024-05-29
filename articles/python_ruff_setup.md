---
title: "Pythonの Linter Formatter は、もうRuff一択。最短5分でプロジェクトに導入" # 記事のタイトル
emoji: "🐍" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["python", "ruff", "linter"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# 概要

Python の Linter、Formatter は今まで black, flake8, isort など複数のライブラリを組み合わせて使用することが一般的でしたが、すべてをオールインワン統合した Ruff の登場により、状況が一変しました。
Ruff は Rust 製の高速な Linter、Formatter で、Python のコードをチェックする際には、もう Ruff 一択と言っても過言ではありません。

本記事では、基本的な Ruff の導入方法(pre-commit を使用)と、VSCODE での設定方法を紹介します。

# リポジトリ

本記事の説明に使用しているサンプルの実装は、以下のリポジトリです。

https://github.com/takashi-yoneya/ruff-example

# 想定読者

Python や Git の基本的な使い方を理解している方を想定しているため、基本的な用語説明は省略しています。

# 環境

エンジニアの利用率の高い macOS を前提として説明していますので、その他の環境の方は随時読み替えてください。
開発環境は VSCODE の前提で説明しています。

# 使用するパッケージ

- ruff
- pre-commit

# Ruff をプロジェクトに導入する

pre-commit は commit 前にコード全体をチェックすることができるライブラリです。
Ruff のような Linter ツールは pre-commit を使用して pre-commit 内で完結してしまった方が、環境に ruff をインストールする必要もなく簡単です。pre-commit がインストールされていない場合は、以下のコマンドでインストールします。

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

# Ruff の設定ファイル

Python では pyproject.toml を使用してプロジェクト全体の設定を管理します。Ruff の設定例を以下に示します。こちらの設定は FastAPI に設定されている ruff の設定を参考にしています。

select にはチェックするエラーの種類を指定します。"ALL"を指定するとすべてのエラーをチェックします。

実行して対処不要なエラーは ignore に追記していくなどして、プロジェクトに合わせて設定を調整していきます。

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

# VSCODE での設定

Ruff では VSCODE 用の以下の拡張機能が提供されており、保存時などに自動て format を適用したり、エラー箇所を強調表示することができます。

```text
名前: Ruff
ID: charliermarsh.ruff
説明: A Visual Studio Code extension with support for the Ruff linter.
バージョン: 2024.22.0
パブリッシャー: Astral Software
VS Marketplace リンク: https://marketplace.visualstudio.com/items?itemName=charliermarsh.ruff
```

以下のように VSCODE の設定ファイルに記述することで、保存時に自動でフォーマットを適用することができます。

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

# まとめ

Ruff の簡単な導入方法を紹介しました。Flake8, black, isort などのライブラリを組み合わせて使用する場合と比較して、設定が簡単で、高速に動作するため、今後の Python プロジェクトでの活用をご検討いただければと思います。
