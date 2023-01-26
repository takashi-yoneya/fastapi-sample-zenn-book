---
title: "FastAPIで、なんとなくasync(非同期)を使用してはいけない、という話" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["fastapi", "python", "backend"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# asyncの安易な選択はNG
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

結論から言うと、asyncを使用するのであれば、そのpath関数から呼ばれる全ての処理をasync対応にしないと、パフォーマンス上の問題が発生する可能性があります。

特に大きいのはDB対応で、sqlalchemyではasync対応もされていますが、Style2.0という方法でクエリを書く必要があり、旧来の方法で実装されている場合は、書き換えには大きな工数がかかります。

# 何が問題なのか？
FastAPIの仕様で、defで作成されたpath関数は、自動的にマルチスレッド化されて平行処理されます。
一方async defで作成されて場合は、単一スレッドとして処理されます。
全ての処理がasync対応されていれば単一スレッドでも、awaitのタイミングで逐次切り替えて平行処理が可能ですが
ブロッキングとなるようなdef関数が間に入ると、awaitのような切り替え処理もされず、さらにマルチスレッドでもないので後続の処理が全てそこで待たされる、という現象が発生します。

コンテナやDBのスペックも問題ないはずなのに、なぜかアクセスが遅くなる時がある

という場合は、この現象が起きている可能性があります。


# 結局はasyncなのかsyncなのか
新規でプロジェクトを始める場合は、asyncで開始するのも良いかと思います。
どのみち１からsqlalchemyのクエリを作るのであれば、手間もあまり変わらないです。

既存のリソースを活かしたいのであれば、syncの方が無難だと思います。

FastAPI対応の有名なpackageのFastAPI-Usersでは、そもそもasyncしか使えない仕様になっています。
そのため、今後はasync前提のpackageも増えてくる可能性があり、マルチスレッドよりはasyncの方が扱いやすいというメリットもあるので
今後をみすえてasyncで構築するのもありだと思います。

