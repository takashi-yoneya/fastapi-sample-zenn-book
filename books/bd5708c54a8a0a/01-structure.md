---
title: "Project構造"
---

# 基本的な考え方

中規模以上の開発に対応するため、役割毎に細かくディレクトリを分けています。


# Project Tree

ディレクトリ、ファイルの構造とそれぞれの役割は以下の通りです。

※ソースコード側は、随時更新していますので、差分がある可能性があります。

```txt
.
|-- Dockerfile.web # Web用コンテナのDockerfile
|-- alembic # migrationを管理
|   |-- README
|   |-- env.py
|   |-- script.py.mako
|   `-- versions # マイグレーションファイル作成を行うと、以下にマイグレーションファイルが作成される
|       |-- 20220318-2304_.py
|       |-- 20220322-2206_.py
|       |-- 20220322-2253_.py
|       |-- 20220323-2155_.py
|       |-- 20220325-0000_.py
|       |-- 20220325-0005_.py
|       |-- 20220403-0857_.py
|       `-- 20220511-1434_.py
|-- alembic.ini # alembicの設定ファイル
|-- app # WebAPI用のメインディレクトリ
|   |-- api # APIエンドポイント用のディレクトリ、配下にv1などのバージョンを切ってもよい
|   |   `-- endpoints
|   |       |-- __init__.py
|   |       |-- categories.py
|   |       |-- develop.py
|   |       |-- jobs.py
|   |       |-- login.py
|   |       |-- tasks.py
|   |       `-- users.py
|   |-- commands # CLIコマンドを定義
|   |   |-- __init__.py
|   |   |-- __set_base_path__.py
|   |   `-- user_creation.py
|   |-- core # 全体で使用するcoreロジックを定義
|   |   |-- __init__.py
|   |   |-- auth.py
|   |   |-- config.py # appの設定を管理する
|   |   |-- database.py # DBセッション等の作成処理
|   |   |-- language_analyzer.py # 言語処理系
|   |   |-- logger # カスタムロガーの定義
|   |   |   |-- __init__.py
|   |   |   |-- logger.py
|   |   |   `-- my_memory_handler.py
|   |   `-- utils.py
|   |-- crud # CRUD処理を定義
|   |   |-- __init__.py
|   |   |-- base.py 
|   |   |-- category.py
|   |   |-- job.py
|   |   `-- user.py
|   |-- exceptions # エラー全般を定義
|   |   |-- __init__.py
|   |   |-- core.py
|   |   |-- error_messages.py # エラーメッセージの定義
|   |   `-- exception_handlers.py 
|   |-- logger_config.yaml # loggerのconfig
|   |-- main.py # FastAPIのmain処理
|   |-- models #SQLAlchemyのmodel
|   |   |-- __init__.py
|   |   |-- base.py
|   |   |-- category.py
|   |   |-- jobs.py
|   |   `-- users.py
|   |-- schemas # Pydanticのスキーマ。主にAPIの入出力スキーマを定義する
|   |   |-- __init__.py
|   |   |-- category.py
|   |   |-- core.py
|   |   |-- job.py
|   |   |-- language_analyzer.py
|   |   |-- request_info.py
|   |   |-- token.py
|   |   `-- user.py
|   `-- tests # テスト全般
|       |-- __init__.py
|       |-- api
|       |   |-- __init__.py
|       |   |-- jobs_test.py
|       |   `-- users_test.py
|       |-- conftest.py
|       |-- core
|       |   |-- __init__.py
|       |   `-- language_analyzer_test.py
|       |-- crud
|       |   |-- __init__.py
|       |   |-- bulk_insert_test.py
|       |   `-- jobs_test.py
|       |-- locust
|       |   `-- locustfile.py
|       |-- rest_client
|       |   `-- elasticsearch.http
|       |-- test_data
|       |   |-- __init__.py
|       |   |-- bulk_data_test.py
|       |   |-- dump.dbml
|       |   |-- dump.sql.gz
|       |   |-- dump.structure.sql
|       |   |-- dump_docker_db_sql.sh
|       |   `-- jobs.py
|       `-- utils
|           |-- test_crud.py
|           |-- test_db_decorator.py
|           |-- user.py
|           `-- utils.py
|-- docker-compose.ecs.yml
|-- docker-compose.es.yml
|-- docker-compose.yml
|-- elasticsearch # Elasticsearch(全文検索エンジンを導入する場合)
|   |-- docker-compose.yml
|   |-- elasticsearch
|   |   `-- Dockerfile.es
|   |-- logstash
|   |   |-- Dockerfile
|   |   `-- pipeline
|   |       `-- main.conf
|   `-- readme.md
|-- poetry.lock # packageのlockファイル
|-- pyproject.toml # poetryの定義ファイル
|-- readme.md
|-- requirements.txt # 実稼働環境などでpoetryを使用しないでpackageをinstallする場合などに使用
`-- runtime.txt # heroku用のruntime設定



```

