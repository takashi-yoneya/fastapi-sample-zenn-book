---
title: "FastAPIでAPIが多くなってくるとSwagger UIがゴチャつくので、機能、ドメイン毎に別々のSwagger UIに分割する方法" # 記事のタイトル
emoji: "🐍" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["python", "fastapi", "swagger"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---


# 概要

FastAPIを使うことでSwagger UIを自動的に生成することができ、APIのドキュメント化や検証に非常に有用ですが、既定では1つのSwagger UIに全てのWebAPIが羅列されるため、規模が大きくなるにつれて、管理が辛くなることが想定されます。

実用案件でFastAPIを使用する場合、APIの数は数百件になることも珍しくないため、機能、ドメイン毎などで別のSwagger UIに分割して管理できた方が幸せになれます。

本記事では、この機能ドメイン等の単位でSwagger UIを分割して管理する方法を紹介します。

なお本記事のソースコードのリンクを含んだ内容を以下のSalesNowのテックブログでも公開しています。

https://tech.salesnow.jp/entry/fastapi-multi-swagger-ui

# FastAPIで複数のSwagger UIを管理する方法

FastAPIはFastAPIクラスを使いappを作成し、このappに任意のPath関数を紐付けていくことでバックエンドを構築することができます。このapp毎にSwagger UIが作成されるため、機能ドメイン等の任意の単位で複数のFastAPIのappを作成し、ベースとなるapp(uvicornで起動するapp)に対して他のappをmountすることで、複数のSwagger UIを作成することができます。

ただし、この状態では、Swagger UI間のリンクが無く不便であるため、以下のコード例ではFastAPIのappのdescriptionに対して、Swagger UIへのリンクを生成するためのHTMLをセットして、Swagger UI上部に他のSwagger UIへのリンクを生成するようにしています。

コード例

```python
# main.py
from fastapi import FastAPI

# appを定義
app = FastAPI(title="App")
feature_app = FastAPI(title="App(/feature)")
admin_app = FastAPI(title="App(/admin)")

# Path関数を定義
@app.get("/")
def base_app_root():
    return {"message": "Hello World"}

@feature_app.get("/")
def feature_app_root():
    return {"message": "Hello World"}

@admin_app.get("/")
def admin_app_root():
    return {"message": "Hello World"}

# BaseとなるAppに他のappをmountする
app.mount("/feature", feature_app)
app.mount("/admin", admin_app)

APP_PATH_LIST = ["", "/feature", "/admin"]

# Swagger間のリンクを作成
def _make_app_docs_link_html(app_path: str, app_path_list: list[str]) -> str:
    """swaggerの上部に表示する各Appへのリンクを生成する"""
    descriptions = [
        f"<a href='{path}/docs'>{path}/docs</a>" if path != app_path else f"{path}/docs"
        for path in app_path_list
    ]
    descriptions.insert(0, "Apps link")
    return "<br>".join(descriptions)

app.description = _make_app_docs_link_html("", APP_PATH_LIST)
feature_app.description = _make_app_docs_link_html("/feature", APP_PATH_LIST)
admin_app.description = _make_app_docs_link_html("/admin", APP_PATH_LIST)
```

コマンド実行例

```bash
# venv作成
$ python -m venv .venv

# venv有効化
$ . .venv/bin/activate

# packageインストール
$ pip install fastapi uvicorn

# uvicornでfastapiを実行
$ python -m uvicorn main:app --port 8080 --reload
```

http://localhost:8080/docsにアクセスすると、以下のような画面が表示されます。

App毎に別のSwagger UIが作成され、その間の相互リンクが画面上部に生成して、ページ間の移動を容易にしています。

baseのAppのSwagger UIです。画面上部にURLリンクが表示されており、これをクリックすることで複数のSwagger UI間を相互に移動することができます。
![base](/images/swagger_base.png)

featureのAppのSwaggerUIです。
![feature](/images/swagger_feature.png)

# 複数のAppを管理する独自クラスを作る

先程の実装でも動作はしますが、Appが多くなった場合に管理が煩雑になる恐れがあるため、FastAPIAppManagerクラスを作成して、管理を容易にしたものは以下の通りです。

複数のappを管理するためのFastAPIAppManagerクラスを作成して、app設定のコピーやSwagger UI間リンクの生成を行っています。

```python
# main2.py （main.pyの改良版）
from fastapi import FastAPI

from app_manager import FastAPIAppManager

# appを定義
app = FastAPI(title="App")
feature_app = FastAPI()
admin_app = FastAPI()
app_manager = FastAPIAppManager(root_app=app)

# Path関数を定義
@app.get("/")
def base_app_root():
    return {"message": "Hello World"}

@feature_app.get("/")
def feature_app_root():
    return {"message": "Hello World"}

@admin_app.get("/")
def admin_app_root():
    return {"message": "Hello World"}

# FastAPIAppManagerにappを追加することで、Linkの自動生成を行う
app_manager.add_app(path="/feature", app=feature_app)
app_manager.add_app(path="/admin", app=admin_app)
app_manager.setup_apps_docs_link()
```

```python
# app_manager.py
from fastapi import FastAPI

class FastAPIAppManager:
    """複数のFastAPI appを管理するためのManagerクラス"""

    def __init__(self, root_app: FastAPI) -> None:
        self.app_path_list: list[str] = [""]
        self.root_app: FastAPI = root_app
        self.apps: list[FastAPI] = [root_app]

    def add_app(self, app: FastAPI, path: str) -> None:
        self.apps.append(app)
        if not path.startswith("/"):
            path = f"/{path}"
        else:
            path = path
        self.app_path_list.append(path)
        # ベースとなるappのTitle等を引き継ぐ
        app.title = f"{self.root_app.title}({path})"
        app.version = self.root_app.version
        app.debug = self.root_app.debug
        self.root_app.mount(path=path, app=app)

    def setup_apps_docs_link(self) -> None:
        """他のAppへのリンクがopenapiに表示されるようにセットする"""
        for app, path in zip(self.apps, self.app_path_list, strict=True):
            app.description = self._make_app_docs_link_html(path)

    def _make_app_docs_link_html(self, current_path: str) -> str:
        """openapiの上部に表示する各Appへのリンクを生成する"""
        descriptions = [
            f"<a href='{path}/docs'>{path}/docs</a>"
            if path != current_path
            else f"{path}/docs"
            for path in self.app_path_list
        ]
        descriptions.insert(0, "Apps link")
        return "<br>".join(descriptions)
```

## ソースコード

上記のソースコードは以下のSalesNowのテックブログ経由で公開しております。

https://tech.salesnow.jp/entry/fastapi-multi-swagger-ui


# まとめ

FastAPIでWebAPIを複数のSwagger UIに分割して管理する方法を紹介しました。

ご意見、ご感想等、コメントにて頂けると嬉しいです。

