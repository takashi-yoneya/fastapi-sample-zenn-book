---
title: "FastAPIがPydantic v2対応したので、V2移行のポイントを紹介する(意外と簡単)" # 記事のタイトル
emoji: "🐍" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["python", "FastAPI", "pydantic"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# 概要
先日、PydanticV2に対応したFastAPI 0.100.0が正式にリリースされました。
PydanticV2は大部分をRustで書き直したことで高速化を実現している他
使い勝手向上のためにAPIが多少変更になっているので、移行作業が必要になる場合があります。

本記事では、V1->V2への移行のポイントについて紹介します。


## 速度向上について
Rust化による速度向上も重要なポイントです。
参考までに、私がPydantic部分のみで試した際は、5~6倍高速化されていました。

以下のクラウドカメラのSafie社のブログで、FastAPIで使用した場合の速度向上について実験されています。

https://engineers.safie.link/entry/2023/07/05/fastapi-pydantic-v2



# 参考リポジトリ
FastAPI 0.100.0に対応したFastAPIのサンプルリポジトリを公開しています。
他にもパッケージ管理のRyeやLinterのRuff、SQLAlchemy v2などの最新のライブラリを適用し、DBはAsync対応しています。

https://github.com/takashi-yoneya/fastapi-template/tree/pydantic-v2-rye

※masterはPydantic v1版のため、上記のpydantic-v2-ryeブランチを参照してください。


# 想定読者
PythonやGitの基本的な使い方を理解している方を想定しているため、基本的な用語説明は省略しています。
また、FastAPIおよびPydanticV1を使用したことがあるを想定しています。

# 環境
エンジニアの利用率の高いmacOSを前提として説明していますので、その他の環境の方は随時読み替えてください。
開発環境はVSCODEの前提で説明しています。
Pythonは3.10を前提とした記述をしています。

# 移行のポイント
## (必須)BaseSettingsが別パッケージのpydantic-settingsになった
アプリ固有の設定を管理するBaseSettingsはV1ではPydanticに含まれていましたが、V2からプラグイン扱いとなり、別のパッケージとなっていますのでinstallおよびimportの変更が必要です。
仕様は殆ど変更がなさそうなので、私はimportの変更だけで対応できました。

V1
```
from pydantic import BaseSettings
```

V2🆕
```
from pydantic_settings import BaseSettings
```

## (必須)既定値がNoneの場合は、=Noneの指定が必須になった
V1では、=Noneを指定しなくても、値が未指定の場合は暗黙にNoneがセットされましたが、Python標準の仕様に合わせて見直され、既定値がNoneの場合は、=Noneの指定が必須となりました。

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

## (必須)validatorの名称が変更
validatorの関数名が変更になり
validator -> field_validator
root_validator -> model_validator
のように、より明確な印象になりました。

また、V1のpre=Trueはmode='before'に変更になっています。modeは'before'以外に'after'も指定可能で、pydanticで型チェック前にvalidateした場合はbeforeを指定します。

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
    def validate_root(cls, values):
        return values

```

## (追加機能)validatorとは別にserializerが追加されjson化される際の変換処理を定義できるようになった
従来は、Pydanticのモデル作成時もシリアライズ時も同一のvalidatorで処理されていましたが、v2からは区別可能になりました。

field_serializer
model_serializer

を使用でき、シリアライズ固有の変換処理などを実装できます。


## (推奨)class Configではなくmodel_config = ConfigDict()を使用する

従来のConfigクラスでは、エディタでの補完やMypyチェックが効かずに、間違っていてもエラーにならない問題がありましたが
v2ではConfigDict()を使用することで、この問題が解消されました。

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

## (推奨)to_camelの標準搭載およびallow_population_by_field_nameがpopulate_by_nameに変更になった
JSONシリアライズ時のキャメルケース変換は、V1では外部ライブラリの追加が必要でしたが、V2ではPydanticに標準搭載されました。
また、configで指定する設定の名称が、allow_population_by_field_nameからpopulate_by_nameに変更になりました。
以下では実用例として、alias_generatorとセットで使用することで、スネークケース、キャメルケースを自動的に相互変換しています。

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


## (推奨)from_ormの非推奨化され、model_validateが新設された
V1では、ORMインスタンスからPydanticインスタンスを作成する場合は、orm_mode=Trueをセットし、from_ormで処理していましたが
V2では、from_attributes=Trueをセットし、model_validateで処理するように変更されています。
ただし、from_ormも現状では従来通り動作します。

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

## (推奨)dict()が非推奨化して、model_dumpが新設された

dict化する処理は、model_dump()が新設されています。

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

フィールド同士の計算によりセットされるフィールドはcomputed_fieldで定義できます。
V1ではroot_validatorなどで実装する場合が多かったですが、よりわかりやすい機能として独立した形です。

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

## (推奨)strict=Trueを指定すると、より厳格に型をチェックできるようになった
strict=Trueを指定すると、str -> intの暗黙的な変換がエラーになるなど、厳密なチェックが実施できます。

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

## (推奨)__fields__が非推奨化され、model_fieldsが新設された
フィールド情報を取得したい場合は、model_fieldsを使用するようになりました。
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