---
title: "Introduction"
---

# はじめに
ご覧いただき、ありがとうございます。
本書では、FastAPIを実務レベルで応用するための、筆者が考えるベストプラクティスを多数紹介致します。

# 対象読者
実務レベルでの応用を想定しているため、以下のように、ある程度バックエンド開発経験がある方を想定しております。
- 何らかのバックエンドフレームワークでの開発経験がある方
- FastAPIの公式リファレンスを参照し、簡単なバックエンドAPIの作成経験がある方
- ORMを使った開発経験のある方

# あまり説明しないこと
- FastAPIの公式リファレンスで説明されていることやPythonの基本事項について
- バックエンドAPIやITの全般的な基本事項について

# Githubリポジトリ
本書で紹介するソースコードは以下で公開しております。
なお、ソースコードは日々Updateしているため、本書の記述内容と差異がある可能性があります。

[https://github.com/takashi-yoneya/fastapi-mybest-template](https://github.com/takashi-yoneya/fastapi-mybest-template)


# 目次
コンテンツは準備中ですが、ソースコードレベルでは実装済のため、すぐに確認したい方は上記のGithubリポジトリのReadmeを参照してください。

## Basic
- Routerを使用したAPIエンドポイントの管理
- DBレコードの作成・取得・更新・削除(CRUD)
- User管理(ログイン、ログアウト、権限)
- パッケージ管理、タスクランナー管理(Package management, task runner management)
- エラーメッセージ、エラーコードの効率的な実装
- configファイルを使用したloggingの実装
- テスト(Testing)
- DBマイグレーション(DB migrations)
- GithubActionsを使用したECSへのデプロイ(CI/CD)

# Additional
- 効率的な論理削除の実装
- キャメルケースとスネークケースの相互変換(Mutual conversion between CamelCase and SnakeCase)
- バッチ処理(Batch)
- Settingsクラスを使用した設定情報の集中管理
- CORS-ORIGINの柔軟な設定方法
- Sentryを使用したログの集中管理(Sentry log management)
- 自然言語解析(sudachi language analyze)