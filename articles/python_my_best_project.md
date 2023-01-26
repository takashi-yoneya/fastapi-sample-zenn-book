---
title: "[2023年最新版]Pythonでプロジェクト作るなら入れるべきパッケージ" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["python"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# リポジトリ

今回説明する内容のリポジトリは以下の通りです。

https://github.com/takashi-yoneya/fastapi-mybest-template

# 何か？
チームでPythonを開発する場合に有用な、汎用的に役立つパッケージを紹介します。

# 構成

## poetry
プロジェクト管理用のパッケージです。
node.jsでいうところのnpmのような位置づけ。

小規模の場合はpython標準のvenvでも問題はないですが、packageのlockファイルや、プロジェクト全体の設定、タスクランナーも一元管理できるので、オススメです。

## poethepoet
poetry用のタスクランナーです。

## mypy
型、構文などの静的チェック。

## black
フォーマッター

## flake8
linter

## autoflake
linter。不使用のimportを削除してくれる

## isort
importの順番を整えてくれる

## pre-commit
commit前にformatやlintを実行できる

## Makefile
Pythonの機能ではないですが、linuxやmacOSの場合はMakefileをタスクランナーとして使用できます。


