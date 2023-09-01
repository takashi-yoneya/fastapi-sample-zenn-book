---
title: "FastAPI Debug Toolbarを使ったRawSQL確認"
---

# なぜ必要か？

SQLAlchemyのようなORMを使う場合、API実行時にどのようなSQLが発行されるかへの意識が薄くなりがちです。
発生しうる代表的な問題としてはN+1問題というものがあります。
記事とタグのように１対Nで紐づいているレコードを取得する際に、複数の記事を取得しつつタグも取得したいような場合に発生することがあります。

# FastAPI Debug Toolbar

FastAPI Debug Toolbarを使うとAPI実行時に、どのようなSQLが発行されたのかや各処理にかかった時間を簡単に確認することができます。

SQL
![SQL](/images/debug_toolbar_sql.png)

HTTPリクエスト全体の時間計測
![Time](/images/debug_toolbar_time.png)

内部処理の時間計測
![Profile](/images/debug_toolbar_profile.png)


# 実装
ORMとしてSQLAlchemyを使用した場合の例を説明します。
fastapi-debug-toolbarのPythonパッケージをインストールすると使用可能になります。
基本的な実装方法としては、main側でappに対してDebugToolbarMiddlewareのmiddlewareを追加することで、使用可能になります。

既定ではSQLAlchemyのSQLはキャプチャされないので、database.py側でengineをaddし、そのPanel
クラスをmain側のpanelsに指定する必要があります。

開発環境以外では表示させたくない場合は、環境によってFastAPIクラスのdebugをFalseに切り替えれば、表示されなくなります。

app/main.py
```python
from fastapi import FastAPI

DEBUG = True

# debug=Trueの場合のみSwaggerにDebugToolbarが表示される
app = FastAPI(
    title="app title",
    debug=DEBUG,
)

# debugモード時はDebugToolbarを有効化する
if DEBUG:
    from debug_toolbar.middleware import DebugToolbarMiddleware

    # panelsに追加で表示するパネルを指定できる
    app.add_middleware(
        DebugToolbarMiddleware,
        panels=["app.core.database.SQLAlchemyPanel"],
    )

```

app/core/database.py
```python
from debug_toolbar.panels.sqlalchemy import SQLAlchemyPanel as BasePanel
from fastapi import Request
from sqlalchemy import MetaData, create_engine
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import Session, sessionmaker
from sqlalchemy.sql import text

from app.main import DEBUG

try:
    async_engine = create_async_engine(
        settings.get_database_url(is_async=True),
        connect_args={"auth_plugin": "mysql_native_password"},
        pool_pre_ping=True,
        echo=False,
        future=True,
    )
    async_session_factory = sessionmaker(
        autocommit=False,
        autoflush=False,
        bind=async_engine,
        class_=AsyncSession,
    )
except Exception as e:
    logger.error(f"DB connection error. detail={e}")

if DEBUG:
    # engineをaddで追加することで、キャプチャ可能になる
    class SQLAlchemyPanel(BasePanel):
        async def add_engines(self, request: Request) -> None:
            self.engines.add(async_engine.sync_engine)
```