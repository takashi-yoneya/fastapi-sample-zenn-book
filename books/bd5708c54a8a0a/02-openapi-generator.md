---
title: "OpenAPI Generatorを使用した、フロントエンド用の型定義、API実行関数の自動生成"
---

# OpenAPI Generator を使うと何が嬉しいか？

Typescrip 以外で実装したバックエンド API の型情報は、フロントエンドと共通できないため、それぞれで重複して定義する必要があり、フロントとバックで型に差異が生まれてしまうなどの問題がありました。

OpenAPI Generator を使用することで、バックエンドの実装から、フロントエンドの型情報を自動的に生成することができます。
また、API 実行用の関数も自動生成できるので、フロントエンドの API 連携の開発体験が最高に高まります。

以下の記事に詳細を記載しております。

https://zenn.dev/tk_resilie/articles/fastapi_openapi_generator_with_react