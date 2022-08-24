---
title: "エンドポイントとCRUDの流れ"
---

# 基本的な考え方

エンドポイント側では、スキーマの入出力処理やCRUDをキックする処理を行い、CRUD側でDBとの連携を行います。

# エンドポイントの実装

以下ではJobというリソースを例として、idを指定してデータを取得するための処理記述しています。


```@app.get```のようにappに直接endpointを紐づけるのではなく```router = APIRouter()```でendpoint毎のrouterを作成し、これを一括でappに紐づけることで、同一endpointのprefixやtag等を一括で管理できます。

[endpoints/jobs.py](https://github.com/takashi-yoneya/fastapi-mybest-template/blob/e1130d4f0437bbfe3bbd2211c832ac0f2107ff67/app/api/endpoints/jobs.py#L25)

```python
from fastapi import APIRouter, Depends

router = APIRouter()

@router.get("/{id}", response_model=schemas.JobResponse)
def get_job(id: str, include_deleted: bool = False, db: Session = Depends(get_db)) -> models.Job:
    job = crud.job.get(db, id=id, include_deleted=include_deleted)
    if not job:
        raise APIException(ErrorMessage.ID_NOT_FOUND)
    return job

app.include_routerで、routerに指定されたendpointを一括で紐づけることができます。

```
[mian.py](https://github.com/takashi-yoneya/fastapi-mybest-template/blob/e1130d4f0437bbfe3bbd2211c832ac0f2107ff67/app/main.py#L49)

```python
app.include_router(jobs.router, tags=["Jobs(求人)"], prefix="/jobs")
```

get_dbでは、sessionの作成、commit、rollbackの管理を行います。

[core/database.py](https://github.com/takashi-yoneya/fastapi-mybest-template/blob/e1130d4f0437bbfe3bbd2211c832ac0f2107ff67/app/core/database.py#L35)
```python
def get_db() -> Generator:
    """
    endpointからアクセス時に、Dependで呼び出しdbセッションを生成する
    エラーがなければ、commitする
    エラー時はrollbackし、いずれの場合も最終的にcloseする
    """
    db = None
    try:
        db = SessionLocal()
        yield db
        db.commit
        ()
    except Exception:
        if db:
            db.rollback()
    finally:
        if db:
            db.close()
```

Schemaのうち、Response系のSchemaはORMから直接Schemaに変換したい場合が多いため、```orm_mode=True```をセットすることで、SQLAlchemyのmodelから直接Schemaに変換できます。
後述するBaseSchemaを継承しており、Schema全体で有効化したい設定は、こちらで定義します。

[schemas/jobs.py](https://github.com/takashi-yoneya/fastapi-mybest-template/blob/e1130d4f0437bbfe3bbd2211c832ac0f2107ff67/app/schemas/job.py#L8)
```python
class JobResponse(BaseSchema):
    id: str
    title: Optional[str]
    hashtags: Optional[List[str]]
    deleted_at: Optional[datetime.datetime]

    class Config:
        orm_mode = True

```

BaseSchemaでは、alias_generator、allow_population_by_field_nameを以下のようにセットすることで、キャメルケース、スネークケースを自動的に変換しています。
これにより、Python側ではスネークケースで管理しつつ、フロントエンド側ではキャメルケースで管理することができるようになります。

[schemas/core.py](https://github.com/takashi-yoneya/fastapi-mybest-template/blob/e1130d4f0437bbfe3bbd2211c832ac0f2107ff67/app/schemas/core.py#L8)
```python
def to_camel(string: str) -> str:
    return camel.case(string)


class BaseSchema(BaseModel):

    class Config:
        """
        # キャメルケース　<-> スネークケースの自動変換
        pythonではスネークケースを使用するが、Javascriptではキャメルケースを使用する場合が多いため
        変換する必要がある
        """

        alias_generator = to_camel
        allow_population_by_field_name = True

```
