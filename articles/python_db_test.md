---
title: "Python(pytest)でテストケース毎にDBを初期化してテストする" # 記事のタイトル
emoji: "🐍" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["python", "pytest", "db"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# 概要


# リポジトリ
本記事の説明に使用しているサンプルのテスト実装は、以下のリポジトリです。

https://github.com/takashi-yoneya/python-test-template

# 想定読者
PythonやGitの基本的な使い方を理解している方を想定しているため、基本的な用語説明は省略しています。

# 環境
エンジニアの利用率の高いmacOSを前提として説明していますので、その他の環境の方は随時読み替えてください。
開発環境はVSCODEの前提で説明しています。

DBはMySQLを想定していますが、PostgreSQLでもほぼ同様の手順で実施可能です。

# 使用するパッケージ
- pytest
- pytest_mysql
- pytest_mock: mockを簡単に管理できるパッケージ

# DBを毎回初期化するためのfixture

engine

```python

```

DBセッション

```python
```


# conftest.pyを使ってテストデータの共通化と個別化を両立する

fixture

# CI(GithubActions)で自動テストする