---
title: "CRUDの実装(データ取得、作成、更新、削除)"
---

# 基本的な考え方
CRUD処理はバックエンドAPIにおける肝となる部分で、殆どの処理に関わってきます。
そのため、可能な限りコードを抽象化することで、個別の実装を最小化します。

# BaseとなるClassの作成

BaseとなるClassを以下のようにCRUDBaseとして定義します。
このClassをベースに、様々なmodelのデータを扱うため、Genericクラスを使用して汎用的に機能するようにしています。

メソッドのget,get_list,update,create,deleteにCRUDのベースとなる処理を実装していきます。

```python
from typing import Any, Generic, Optional, Type, TypeVar

from pydantic import BaseModel
from sqlalchemy.orm import Session, query

# ユーザー定義タイプ
ModelType = TypeVar("ModelType", bound=Base)
CreateSchemaType = TypeVar("CreateSchemaType", bound=BaseModel)
UpdateSchemaType = TypeVar("UpdateSchemaType", bound=BaseModel)
ListResponseSchemaType = TypeVar("ListResponseSchemaType", bound=BaseModel)


class CRUDBase(Generic[ModelType, CreateSchemaType, UpdateSchemaType, ListResponseSchemaType]:
    
    def __init__(self, model: Type[ModelType]) -> None:
        self.model = model

    def get(self, db: Session, id: Any, include_deleted_data: bool = False) -> Optional[ModelType]:
        pass
    
    def get_list(
        self,
        db: Session,
        filtered_query: Optional[query.Query] = None,
        include_deleted_data: bool = False) -> list[ModelType]:
        pass

    def create(self, db: Session, obj_in: CreateSchemaType) -> ModelType:
        pass

    def update(self, db: Session, *, db_obj: ModelType, obj_in: UpdateSchemaType) -> ModelType:
        pass

    def delete(self, db: Session, db_obj: ModelType) -> ModelType:
        pass

```

