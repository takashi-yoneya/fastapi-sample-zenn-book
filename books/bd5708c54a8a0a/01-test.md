---
title: "Test"
---

# 基本的な考え方

テストケース毎に環境を初期化することで、冪等性の高いテストを実装する方法を紹介します。
Pythonの一般的なテストライブラリであるpytestを使用しています。


# Test(pytest)の基本事項

pytestの基本的な事項については、以下の記事を参考にしてください。
[pytestの基本事項](https://zenn.dev/tk_resilie/articles/python_test_template)

# DBの初期化

テストケース毎にDBの状態を初期化するためには、pytest_mysql(pytest_posgresもある)を使用します。
これにより、テストケース毎にDBの状態を初期化することができます。

asyncの場合の処理を記述していますが、sync処理とすることも容易です。

以下をtestルートのconftest.pyに記述しておくと、全てのテストでfixtureとしてdbが使用可能になります。

conftest.py
```python

# pytest_mysqlを使って、テストで使用するDBの接続先設定等を行う
db_proc = factories.mysql_noproc(
    host=DB_HOST,
    port=DB_PORT,
    user=DB_USER_NAME,
)
mysql = factories.mysql("db_proc")

@pytest_asyncio.fixture
async def engine(
    mysql: Any,  # noqa: ARG001
) -> AsyncEngine:
    """fixture: db-engineの作成およびmigrate"""
    uri = settings.get_database_url(is_async=True)
    engine = create_async_engine(uri, echo=False, poolclass=NullPool)
    # create_allで一括でDBテーブルを定義する
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    return engine


@pytest_asyncio.fixture
async def db(
    engine: AsyncEngine,
) -> AsyncGenerator[AsyncSession, None]:
    """fixture: db-sessionの作成"""
    test_session_factory = sessionmaker(autocommit=False, autoflush=False, bind=engine, class_=AsyncSession)

    # DBセッションの作成
    async with test_session_factory() as session:
        yield session
        # セッション終了後にはcommitする
        await session.commit()

```

# Test用のHTTPクライアントの定義

```app.dependency_overrides[get_async_db] = override_get_db```のように記述することで、APIのDependsで指定するget_aysnc_dbをテスト用の定義でoverrideできます

```python

@pytest_asyncio.fixture
async def client(engine: AsyncEngine) -> AsyncClient:
    """fixture: HTTP-Clientの作成"""
    logger.debug("fixture:client")
    test_session_factory = sessionmaker(autocommit=False, autoflush=False, bind=engine, class_=AsyncSession)

    async def override_get_db() -> AsyncGenerator[AsyncSession, None]:
        async with test_session_factory() as session:
            yield session
            await session.commit()

    # get_dbをTest用のDBを使用するようにoverrideする
    app.dependency_overrides[get_async_db] = override_get_db
    app.debug = False
    return AsyncClient(app=app, base_url="http://test")

```