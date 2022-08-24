---
title: "論理削除(soft delete)"
---

# 基本的な考え方

DBからレコードを削除する場合は、影響を最小限にするために、物理的な削除ではなく、論理的な削除を行う実装が良く使用されますが、これを個々のCRUDで実装してしまうと、実装漏れにより削除したはずのレコードが表示されてしまう問題が発生します。
そのため、論理削除は１箇所のみで実装する方針として実装しています。

# event-listenerを使用した実装

```@event.listens_for(Session, "do_orm_execute")```を使用することで、全てDBのセッションでORMの実行直前に行う処理を定義することができます。

以下のサンプルでは、deleted_at(削除日時)で削除済かどうかを判定しています。
orm.with_loader_criteriaを使用することで、任意のModelに追加のfilterを定義できます。
そこで、ModelBaseを定義し、deleted_atを含む全てのModelで共通的に使用するColumnを定義しておきます。
その上で、orm.with_loader_criteriaの第一引数にModelBaseを指定することで、ModelBaseを継承している全てのModelに、自動的に```deleted_at.is_(None)```が追加されることで、論理削除済レコードを除外することができます。

この方法はSQLAlchemy公式でも推奨されています。
https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.with_loader_criteria

論理削除済のレコードを取得したい場合にも対応するため、```and not execute_state.execution_options.get("include_deleted", False)```の行で制御しています。
```query(...).filter(...).execution_options(include_deleted=True)```のように```execution_options(include_deleted=True)```を含むqueryを作成することで、論理削除filter処理をスキップできます。

```python
from core.logger import get_logger
from core.utils import get_ulid
from sqlalchemy import Column, DateTime, String, event, orm
from sqlalchemy.orm import Session
from sqlalchemy.sql.functions import current_timestamp

logger = get_logger(__name__)


class ModelBase:
    id = Column(String(32), primary_key=True, default=get_ulid)
    created_at = Column(DateTime, nullable=False, server_default=current_timestamp())
    deleted_at = Column(DateTime)


@event.listens_for(Session, "do_orm_execute")
def _add_filtering_deleted_at(execute_state):
    """
    論理削除用のfilterを自動的に適用する
    以下のようにすると、論理削除済のデータも含めて取得可能
    query(...).filter(...).execution_options(include_deleted=True)
    """
    if (
        execute_state.is_select
        and not execute_state.is_column_load
        and not execute_state.is_relationship_load
        and not execute_state.execution_options.get("include_deleted", False)
    ):
        execute_state.statement = execute_state.statement.options(
            orm.with_loader_criteria(
                ModelBase,
                lambda cls: cls.deleted_at.is_(None),
                include_aliases=True,
            )
        )

```