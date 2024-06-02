---
title: "FastAPIがPydantic v2対応したので、V2移行のポイントを紹介する(意外と簡単)" # 記事のタイトル
emoji: "🐍" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["python", "FastAPI", "pydantic"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# 概要

先日、PydanticV2 に対応した FastAPI 0.100.0 が正式にリリースされました。
PydanticV2 は大部分を Rust で書き直したことで高速化を実現している他
使い勝手向上のために API が多少変更になっているので、移行作業が必要になる場合があります。

本記事では、V1->V2 への移行のポイントについて紹介します。

## 速度向上について

Rust 化による速度向上も重要なポイントです。
参考までに、私が Pydantic 部分のみで試した際は、5~6 倍高速化されていました。

以下のクラウドカメラの Safie 社のブログで、FastAPI で使用した場合の速度向上について実験されています。

https://engineers.safie.link/entry/2023/07/05/fastapi-pydantic-v2

# 参考リポジトリ

FastAPI 0.100.0 に対応した FastAPI のサンプルリポジトリを公開しています。
他にもパッケージ管理の Rye や Linter の Ruff、SQLAlchemy v2 などの最新のライブラリを適用し、DB は Async 対応しています。

https://github.com/takashi-yoneya/fastapi-template/tree/pydantic-v2-rye

※master は Pydantic v1 版のため、上記の pydantic-v2-rye ブランチを参照してください。

# 想定読者

Python や Git の基本的な使い方を理解している方を想定しているため、基本的な用語説明は省略しています。
また、FastAPI および PydanticV1 を使用したことがあるを想定しています。

# 環境

エンジニアの利用率の高い macOS を前提として説明していますので、その他の環境の方は随時読み替えてください。
開発環境は VSCODE の前提で説明しています。
Python は 3.10 を前提とした記述をしています。

# 移行のポイント

## (必須)BaseSettings が別パッケージの pydantic-settings になった

アプリ固有の設定を管理する BaseSettings は V1 では Pydantic に含まれていましたが、V2 からプラグイン扱いとなり、別のパッケージとなっていますので install および import の変更が必要です。
仕様は殆ど変更がなさそうなので、私は import の変更だけで対応できました。

V1

```
from pydantic import BaseSettings
```

V2🆕

```
from pydantic_settings import BaseSettings
```

## (必須)既定値が None の場合は、=None の指定が必須になった

V1 では、=None を指定しなくても、値が未指定の場合は暗黙に None がセットされましたが、Python 標準の仕様に合わせて見直され、既定値が None の場合は、=None の指定が必須となりました。

V1

```python
class TodoResponse(BaseModel):
    id: str
    title: str
    created_at: datetime.datetime
    updated_at: datetime.datetime | None #=Noneなしでも、値未指定ならNoneとみなされた
```

V2🆕

```python
class TodoResponse(BaseModel):
    id: str
    title: str
    created_at: datetime.datetime
    updated_at: datetime.datetime | None = None # =Noneがない場合は、値の指定が必須になった
```

## (必須)validator の名称が変更

validator の関数名が変更になり
validator -> field_validator
root_validator -> model_validator
のように、より明確な印象になりました。

また、V1 の pre=True は mode='before'に変更になっています。mode は'before'以外に'after'も指定可能で、pydantic で型チェック前に validate した場合は before を指定します。

以下の例は[bump-pydantic](https://github.com/pydantic/bump-pydantic)より転記しています。

V1

```python
from pydantic import BaseModel, validator, root_validator


class User(BaseModel):
    name: str

    @validator('name', pre=True) # <-
    def validate_name(cls, v):
        return v

    @root_validator(pre=True) # <-
    @classmethod
    def validate_root(cls, values):
        return values
```

V2🆕

```python
from pydantic import BaseModel, field_validator, model_validator


class User(BaseModel):
    name: str

    @field_validator('name', mode='before') # <-
    def validate_name(cls, v):
        return v

    @model_validator(mode='before') # <-
    @classmethod
    def validate_root(cls, values):
        return values

```

### model_validator で mode を after に指定した場合と before に指定した場合の違い

before を指定した場合は Pydantic インスタンス作成前時点で動作するため、入力値は dict 形式のデータになります。この dict 形式のデータに任意のバリデーションを行います。レスポンスは dict 形式で返す必要があります。
after を指定した場合は Pydantic インスタンス作成後に動作するため、作成したインスタンスに対してバリデーションを行います。レスポンスはインスタンス自身(self)を返す必要があります。

```python

from typing import Self, Any

class User(BaseModel):
    name: str

    # インスタンス作成前に動作するためクラスメソッドになる
    @model_validator(mode='before')
    @classmethod
    def validate_root_before(cls, values: dict[str, Any]) -> dict[str, Any]:
        return values

    # 作成されたインスタンスに対する操作のためインスタンスメソッドになる
    @model_validator_after(mode='after')
    def validate_root(self) -> Self:
        return self

```

## (追加機能)validator とは別に serializer が追加され json 化される際の変換処理を定義できるようになった

従来は、Pydantic のモデル作成時もシリアライズ時も同一の validator で処理されていましたが、v2 からは区別可能になりました。

field_serializer
model_serializer

を使用でき、シリアライズ固有の変換処理などを実装できます。

## (推奨)class Config ではなく model_config = ConfigDict()を使用する

従来の Config クラスでは、エディタでの補完や Mypy チェックが効かずに、間違っていてもエラーにならない問題がありましたが
v2 では ConfigDict()を使用することで、この問題が解消されました。

V1

```python
class TodoResponse(BaseModel):
    id: str

    class Config:
      ...　# 設定を記述
```

V2🆕

```python
from pydantic import ConfigDict

class TodoResponse(TodoBasBaseModele):
    id: str

    model_config = ConfigDict(...) # 設定を記述
```

## (推奨)to_camel の標準搭載および allow_population_by_field_name が populate_by_name に変更になった

JSON シリアライズ時のキャメルケース変換は、V1 では外部ライブラリの追加が必要でしたが、V2 では Pydantic に標準搭載されました。
また、config で指定する設定の名称が、allow_population_by_field_name から populate_by_name に変更になりました。
以下では実用例として、alias_generator とセットで使用することで、スネークケース、キャメルケースを自動的に相互変換しています。

V2🆕

```python
from pydantic import BaseModel, ConfigDict
from pydantic.alias_generators import to_camel # pydanticに標準搭載された

class BaseSchema(BaseModel):
    """全体共通の情報をセットするBaseSchema"""

    # class Configで指定した場合に引数チェックがされないため、ConfigDictを推奨
    model_config = ConfigDict(
      alias_generator=to_camel,
      populate_by_name=True, # V1: allow_population_by_field_name=True
    )

```

## (推奨)from_orm の非推奨化され、model_validate が新設された

V1 では、ORM インスタンスから Pydantic インスタンスを作成する場合は、orm_mode=True をセットし、from_orm で処理していましたが
V2 では、from_attributes=True をセットし、model_validate で処理するように変更されています。
ただし、from_orm も現状では従来通り動作します。

V2🆕

```python
class TodoResponse(TodoBase):
    id: str
    tags: list[TagResponse] | None = []
    created_at: datetime.datetime | None = None
    updated_at: datetime.datetime | None = None

    model_config = ConfigDict(from_attributes=True)　# V1: from_mode=True

# ormインスタンスからpydanticインスタンスを生成
TodoResponse.model_validate(orm_obj) # V1: TodoResponse.from_orm(orm_obj)

```

## (推奨)dict()が非推奨化して、model_dump が新設された

dict 化する処理は、model_dump()が新設されています。

V2🆕

```python
class TodoResponse(TodoBase):
    id: str
    tags: list[TagResponse] | None = []
    created_at: datetime.datetime | None = None
    updated_at: datetime.datetime | None = None

    model_config = ConfigDict(from_attributes=True)　# V1: from_mode=True

# ormインスタンスからpydanticインスタンスを生成
data = TodoResponse.model_validate(orm_obj)
data.model_dump() # dict化される
```

## (新機能)computed_field

フィールド同士の計算によりセットされるフィールドは computed_field で定義できます。
V1 では root_validator などで実装する場合が多かったですが、よりわかりやすい機能として独立した形です。

以下のコードは公式サイトからの引用です。

V2🆕

```python
from pydantic import BaseModel, computed_field


class Rectangle(BaseModel):
    width: int
    length: int

    @computed_field
    @property
    def area(self) -> int:
        return self.width * self.length


print(Rectangle(width=3, length=2).model_dump())
#> {'width': 3, 'length': 2, 'area': 6}
```

## (推奨)strict=True を指定すると、より厳格に型をチェックできるようになった

strict=True を指定すると、str -> int の暗黙的な変換がエラーになるなど、厳密なチェックが実施できます。

詳細は以下の公式サイトで確認できます。
https://docs.pydantic.dev/latest/usage/strict_mode/

V2🆕

```python
class BaseSchema(BaseModel):
    """全体共通の情報をセットするBaseSchema"""

    model_config = ConfigDict(
      strict=True
    )
```

## (推奨)**fields**が非推奨化され、model_fields が新設された

フィールド情報を取得したい場合は、model_fields を使用するようになりました。
以下の例では、フィールド名の一覧取得しています。

V1

```python
list(TodoResponse.__fields__.keys())
```

V2🆕

```python
list(TodoResponse.model_fields.keys())
```

# 移行ツール

公式から移行ツールが提供されています。
当方は未検証ですが、コード量が多い場合は有用だと思われます。

[bump-pydantic](https://github.com/pydantic/bump-pydantic)
