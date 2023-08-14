---
title: "FastAPIでDBが非同期対応(async)していない場合は、router(Path関数)でasyncを使用してはいけない" # 記事のタイトル
emoji: "🐍" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["fastapi", "python", "backend"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# FastAPIのrouterにはasync def/defのいずれも指定できるが、適当に決めてはならない
FastAPIのrouter定義(path関数)では、def/async def のいずれも使用できるため、サンプルなどを見てなんとなくasync defにしている方もいるのではないでしょうか？

以下にasyncとsyncの実装例を記述します。

```
# sync
@router.get("/sync")
def get_develop_sync(i: int) -> None:
    import time
    print(f"start: {i}")
    time.sleep(30)
    print(f"end: {i}")


# async
@router.get("/async")
async def get_develop_async(i: int) -> None:
    import time
    import asyncio
    print(f"async start: {i}")
    asyncio.sleep(30)
    print(f"async end: {i}")
```

結論から言うと、asyncを使用するのであれば、そのPath関数から呼ばれる全ての処理をasync対応にしないと、パフォーマンス上の問題が発生する可能性があります。

特に大きいのはDBアクセスで、sqlalchemyではasync対応もされていますが、Style2.0という方法でクエリを書く必要があり、旧来の方法で実装されている場合は、書き換えには大きな工数がかかります。
よって現段階では、同期関数で定義した方が良い場合が多いと思います。

# router(Path関数)にasyncを使用してしまうと、DBでブロッキングされてパフォーマンス悪化要因になる
defとasync defの場合で、以下のようなFastAPIの内部処理の違いがあるため、async defかつDBが同期処理の場合は、問題になる可能性が高いです。

## defで定義した場合
defで定義されたPath関数は、FastAPIが内部的にマルチスレッド化して実行します。そのため同時にリクエストされても、ブロッキングされずにマルチスレッドで処理可能な数までは同時処理可能です。

## async defで定義した場合
async defで定義されたPath関数は、FastAPIはシングルスレッドの非同期処理(async)として実行するため、通常のasync関数と同様に処理の途中にdef関数が挟まるとブロッキング要素となり、同時処理されなくなります。
そのため、DBアクセスのような時間がかかる処理が同期処理として記述されている場合は、ブロッキングによる待ち時間が多くなり、全体としてのパフォーマンスが低下します。

# 結局はasyncなのかsyncなのか
DBアクセスをはじめとする時間のかかりうる処理がasync対応できているのであれば、routerもasync defで定義しても良いと思います。
ただ、よくわからない場合やasync対応の知見が乏しい場合は、defで定義した方が無難です。

