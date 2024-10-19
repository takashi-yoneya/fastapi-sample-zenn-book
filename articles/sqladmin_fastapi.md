---
title: "FastAPIに簡単に管理画面を追加できるSQlAdminが凄い" # 記事のタイトル
emoji: "🐍" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["python", "admin", "sqladmin", "fastapi"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
publication_name: where
---

# はじめに・概要

株式会社 Penetrator で業務委託でエンジニアをしている米谷([@yoneya_fastapi](https://x.com/yoneya_fastapi))です。

Penetrator では、衛星データを活用して不動産情報の自動取得を可能にするアプリ「WHERE（ウェアー）」を開発、提供しており、バックエンドの開発には Python(フレームワーク: FastAPI) を採用しています。

FastAPI は簡単にバックエンド API が作成できますが、Django のような管理画面機能がないため、管理画面を作成するためには別途実装する必要があります。
DB テーブル毎に CRUD 画面を作成するのは手間がかかるため、SQLAdmin というライブラリを使用することで簡単に管理画面を追加することができます。
Python の有名な ORM である SQLAlchemy に対応しており、最小限のコード追加で管理画面を追加することができます。

本記事では、FastAPI に SQLAdmin を導入する方法と、簡単な CRUD 画面の作成方法を紹介します。

# リポジトリ

本記事の説明に使用しているサンプルの実装は、以下のリポジトリを参照してください。

https://github.com/takashi-yoneya/fastapi-sqladmin

# 想定読者

Python や Git の基本的な使い方を理解している方を想定しているため、基本的な用語説明は省略しています。
また SQLAlchemy についての説明は省略しています。

# 環境

エンジニアの利用率の高い macOS を前提として説明していますので、その他の環境の方は随時読み替えてください。
開発環境は VSCode の前提で説明しています。

# 使用する主なライブラリ

- FastAPI (Python の Web フレームワーク)
- SQLAdmin (FastAPI に管理画面を追加するライブラリ)
- SQLAlchemy (Python の ORM ライブラリ)

# SQLAdmin の説明

## 依存パッケージのインストール

今回は FastAPI, SQLAlchemy, SQLAdmin を使用するため、以下のコマンドでインストールします。なお、SQLAdmin は SQLModel にも対応しています。

```bash
# uv でインストールする場合
uv add sqladmin fastapi sqlalchemy

# pip でインストールする場合
pip install sqladmin fastapi sqlalchemy
```

## ディレクトリ構造の例

今回の例では、app ディレクトリ配下に管理画面用のファイルを格納しています。
admin_main.py を起動ファイルとし、admin ディレクトリに templates や view などの管理画面用のファイルを格納しています。
管理画面以外のバックエンド API も同一のプロジェクトで管理することを想定して main.py を用意しています。

```pre
├── app
│   ├── admin # 管理画面用のファイルを格納
│   │   ├── core
│   │   │   └── auth.py
│   │   ├── templates # Jinja2テンプレートを格納
│   │   │   └── user_csv_import.html
│   │   └── views # Viewを格納
│   │       ├── tags.py
│   │       ├── todos.py
│   │       ├── todos_tags.py
│   │       ├── user_csv_import.py
│   │       └── users.py
│   ├── admin_main.py # 管理画面のメイン処理
│   ├── database.py
│   ├── logger_config.yaml
│   ├── main.py # FastAPIのメイン処理(管理画面だけを構築する場合は不要)
│   ├── models # SQLAlchemyのModelを格納
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── tags.py
│   │   ├── todos.py
│   │   ├── todos_tags.py
│   │   └── users.py
│   └── settings.py
```

## メインファイルの作成

admin_main.py(名称は何でも良い)は管理画面のメイン処理を記述するファイルです。以下のように記述します。
SQLAdmin 自体は Web サーバーの機能を持たないため、FastAPI を使用して Web サーバーを起動し、SQLAdmin 用のエンドポイントを追加する方式になります。

以下ソースコードでは、aの箇所 で FastAPI アプリケーションを作成し、bの箇所 で SQLAdmin のインスタンスを作成します。cの箇所 で管理画面の view を追加しています。

admin_main.py

```python
import logging

from fastapi import FastAPI
from sqladmin import Admin

from app.admin.core.auth import AdminAuth
from app.admin.views.tags import TagAdminView
from app.admin.views.todos import TodoAdminView
from app.admin.views.todos_tags import TodoTagAdminView
from app.admin.views.user_csv_import import UserCsvImportAdminView
from app.admin.views.users import UserAdminView
from app.database import async_session_factory
from app.settings import settings

""" 管理画面用のmain処理 """

# loggingセットアップ(任意)
logger = logging.getLogger(__name__)


# a.管理画面用のFastAPIアプリケーションの作成
app = FastAPI(
    title=settings.TITLE,
    version=settings.VERSION,
    debug=settings.DEBUG or False,
)

# b.管理画面用クラスの作成
admin_manager = Admin(
    app,
    title="管理画面",
    session_maker=async_session_factory,  # SQLAlchemyのセッションを作成する関数
    templates_dir="app/admin/templates", # Jinja2テンプレートのディレクトリ
    # 認証用のクラスを指定
    authentication_backend=AdminAuth(secret_key=settings.SECRET_KEY),
)


# c.管理画面のviewを追加する
admin_manager.add_view(UserAdminView)
admin_manager.add_view(TodoAdminView)
admin_manager.add_view(TagAdminView)
admin_manager.add_view(TodoTagAdminView)
admin_manager.add_view(UserCsvImportAdminView)


```

管理画面用クラスの作成部分を詳細に説明します。

session_maker には SQLAlchemy のセッションを作成する関数を指定します。今回は async 対応のセッションを作成するため、async_session_factory を指定しています。
渡された session_maker は、各 view で ORM に渡されて使用されます。

templates_dir には Jinja2 テンプレートのディレクトリを指定します。SQLAdmin は Jinja2 テンプレートを使用して管理画面の HTML を作成します。
SQLAdmin 標準の template だけを使用する場合は、特に指定する必要はありませんが、任意の HTML を作成したい場合は、templates_dir にディレクトリを指定します。

authentication_backend には認証用のクラスを指定します。SQLAdmin は認証用のクラスを指定することで、管理画面にアクセスする際のログイン等の認証を行うことができます。
基本的には AuthenticationBackend クラスを継承した独自の認証用クラスを作成し、認証処理を記述します。（認証部分の説明については後述）

```python
# b.管理画面用クラスの作成
admin_manager = Admin(
    app,
    title="管理画面",
    session_maker=async_session_factory,  # SQLAlchemyのセッションを作成する関数
    templates_dir="app/admin/templates", # Jinja2テンプレートのディレクトリ
    # 認証用のクラスを指定
    authentication_backend=AdminAuth(secret_key=settings.SECRET_KEY),
)
```

## 認証用クラスの作成

認証用クラスは AuthenticationBackend クラスを継承して作成します。以下のように記述します。
login、logout、authenticate メソッドをオーバーライドして実装することで、管理画面にアクセスする際のログイン、ログアウト、認証処理を記述することができます。
認証用クラスを設定するとログイン画面が自動的に作成され、ユーザー名、パスワードを入力してログインすることができます。ここで入力されたユーザー名、パスワードは login メソッドに渡さるので、これを使って任意の認証処理を記述し、成功したら token をセットするようにします。
logout メソッドはログアウトボタンを押下した際の処理を記述します。ここでは token を削除するようにします。

この実装はあくまで一例のため、プロジェクトに合わせて柔軟にカスタマイズすることが可能です。

```python
from sqladmin.authentication import AuthenticationBackend
from starlette.requests import Request


class AdminAuth(AuthenticationBackend):
    async def login(self, request: Request) -> bool:
        """ログインボタンを押下した際の処理"""
        # フォームで入力したユーザ名とパスワードを取得
        form = await request.form()
        username, password = form["username"], form["password"]

        # ここに任意のチェック処理を記述
        # ...

        request.session["token"] = "dummy_token"

        return True

    async def logout(self, request: Request) -> bool:
        """ログアウトボタンを押下した際の処理"""
        request.session.clear()
        return True

    async def authenticate(self, request: Request) -> bool:
        """認証処理（権限有無の確認）"""
        token = request.session.get("token")

        if not token:
            return False

        # ここに任意のチェック処理を記述
        # ...

        return True
```

## ModelView(ORM 連携 View) の作成

ModelView を使用すると ORM と連携して指定した Model(テーブル)に対する CRUD 画面を作成できます。
以下のように ModelView を継承して作成し、model に ORM の Model クラスを指定します。name_plural に表示名、column_list に表示するカラム、column_searchable_list に検索可能なカラム、form_columns にフォームに表示するカラムを指定します。

```python
from sqladmin import ModelView

from app.models import Tag


class TagAdminView(ModelView, model=Tag):  # type: ignore
    name_plural = "Tags" # 表示名
    column_list = "__all__" # すべてのカラムを表示
    column_searchable_list = [Tag.name] # 検索可能なカラムを指定
    form_columns = [Tag.name] # フォーム(Create, Edit)に表示するカラムを指定
```

ここで作成した View クラスは、先述の admin_main.py のように`add_view`メソッドで管理画面に追加すると管理画面に表示されるようになります。

```python
admin_manager.add_view(TagAdminView)
```

![tags list](/images/sqladmin_tags_list.png)

## BaseView(カスタム View) の作成

１つの Model に紐付かないような管理画面や通常の CRUD 画面以外の画面を作りたい場合は BaseView を使用します。
以下の例では、ユーザー CSV 一括登録画面を作成しています。
expose したメソッドがエンドポイントになります。任意の処理を記述できるので、CSV のインポート処理のような柔軟な画面を作成することができます。
この場合は独自に定義した HTML を返すため、templates ディレクトリに HTML ファイルを配置しています。

```python
import logging

from fastapi import Request
from sqladmin import BaseView, expose

logger = logging.getLogger(__name__)


class UserCsvImportAdminView(BaseView):  # type: ignore
    name_plural = "ユーザーCSV一括登録"
    identity = "user_csv_import"
    methods = ["GET", "POST"]

    @expose(f"/{identity}", methods=["GET", "POST"])
    async def user_csv_import(self, request: Request):
        """/user_csv_import のエンドポイント"""
        # GETとPOSTで処理を分ける
        if request.method == "GET":
            # HTMLをJinja2テンプレートを使って作成してレスポンス
            # HTMLの作成に必要な情報はcontextにdictで渡す
            return await self.templates.TemplateResponse(
                request,
                name=f"{self.identity}.html",
                context={"view": self},
            )
        elif request.method == "POST":
            # CSVインポート処理を記述

            # HTMLをJinja2テンプレートを使って作成してレスポンス
            return await self.templates.TemplateResponse(
                request,
                name=f"{self.identity}.html",
                context={
                    "view": self,
                    "success_count": 1,
                    "error_count": 0,
                    "description": "ダミーレスポンスです",
                },
            )
```

上記コードの self.templates.TemplateResponse の name に指定した名称で html ファイルを作成します。
context に渡した変数は html ファイル内で使用できます。view に self を渡すことで、name_plural などを参照できるようにしています。
また POST 後のレスポンスでは処理結果を表示するために、success_count, error_count, description などを渡しています。

templates は、SQLAdmin 標準のものがインストールしたライブラリのディレクトリ(sqladmin/templates)に格納されているため、参考にしてみると良いと思います。

templates/user_csv_import.html

```html
{% extends "sqladmin/layout.html" %} {% block content %}
<div class="col-12">
  <div class="card">
    <div class="card-header">
      <h3 class="card-title">{{ view.name_plural }}</h3>
    </div>
    <div class="card-body border-bottom py-3">
      <h3>
        アップロードするCSVを選択してください。複数ファイルを同時に指定できます。
      </h3>
      <form
        action="/admin/{{view.identity}}"
        method="POST"
        enctype="multipart/form-data"
      >
        <fieldset class="form-fieldset">
          <input
            multiple="multiple"
            type="file"
            name="files"
            id="files"
            accept=".csv"
          />
        </fieldset>
        <div class="col-md-6">
          <div class="btn-group flex-wrap" data-toggle="buttons">
            <input type="submit" name="save" value="登録" class="btn" />
          </div>
        </div>
        <p></p>
        <div class="row">
          {% if success_count %}
          <div class="alert alert-success" role="alert">{{ description }}</div>
          <div class="alert alert-success" role="alert">
            {{ success_count }}件登録されました（重複も含む）
          </div>
          {% endif %} {% if error_count %}
          <div class="alert alert-danger" role="alert">
            {{ error_count }}件のエラーが発生しました
          </div>
          {% if error_descriptions %}
          <div class="alert alert-danger" role="alert">
            エラー詳細<br />
            {% for error_description in error_descriptions %} {{
            error_description }}<br />
            {% endfor %}
          </div>
          {% endif %} {% endif %}
        </div>
      </form>
    </div>
  </div>
</div>
{% endblock %}
```

## 起動方法

FastAPI 上で管理画面を Mount しているため、FastAPI を起動することで管理画面が表示できるようになります。
app/admin_main.py に記述している場合は以下のコマンドで起動し、http://localhost:8000/admin にアクセスすることで管理画面にアクセスできます。

```bash

uvicorn app.admin_main:app --reload

```

ログイン画面が表示されますが、今回の実装では Username, Password は空でもログインすることができます。

![sqladmin login](/images/sqladmin_login.png)

ログイン後、以下のように登録した View が左の一覧に表示されます。

![sqladmin index](/images/sqladmin_index.png)

Todo を選択した場合の一覧画面です。右上の New Todo から新しい Todo を登録することができます。
また検索ボックスにキーワードを入力することで、一覧の絞り込みが可能です。
目のアイコンで詳細を確認、鉛筆アイコンで編集、ゴミ箱アイコンで削除ができます。

![sqladmin list](/images/sqladmin_list.png)

# まとめ

SQLAdmin を使用して FastAPI プロジェクトに簡単に管理画面を追加する方法を紹介しました。
管理画面は基本的に社内メンバーが使用するため、あまり工数をかけずに、必要な機能を追加できることが重要なため、SQLAdmin は非常に便利なライブラリだと思います。
特にスタートアップ企業では、顧客価値に直結する機能の構築にリソースを集中させたい場合が多いと思いますので、管理画面のような運用機能はライブラリを使用することで効率的に実装することで、開発リソースを他の機能に集中できると思います。

私たちは現在、「宇宙から地球の不動産市場を変える」挑戦に共感し、一緒に「おっ！」をつくる情熱的なエンジニアを募集しています。新しい技術に挑戦し、人々を驚かせるプロダクトを生み出したい方は、ぜひ [採用ページ](https://pntwhere-recruit.studio.site/) をご覧ください。
[![where](/images/where_image.png)](https://pntwhere-recruit.studio.site/)
