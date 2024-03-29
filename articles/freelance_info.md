---
title: "フリーランスなる前に知っておくべきこと" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["freelance"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# なぜこの記事を書いたか？
フリーランスになって４年経ち、あらためて纏めます。
これからフリーランスを検討してる方の参考になれば幸いです。
技術レベルについての記事は多いので、ここでは技術以外の部分についても言及していきます。
私がバックエンドを中心としたタスクを行なっているので、バックエンド向きの内容になっています。

# 技術スキル

# フリーランスに向いている人、サラリーマンの方が向いている人
技術が好き。キャッチアップが苦ではない。
コミュニケーションが苦ではない。
主体的に行動できる。指示待ちではない。
自分で手を動かすことが好き

# こころがまえ
## ポジショニングを意識する
市場の中での自分のポジショニングは常に意識する必要がある
今何の技術が求められているか、貪欲に情報収集して
自分のポジショニングを定めておくと良いです。

私であれば、PythonフレームワークのFastAPIを全力やりたい思いがあったので
Twitterの名前にもFastAPIを含めています。
また、MENTAやストアカなどの教育系プラットフォームにも登録することで
自分の名前と得意技術が検索にHITしやすい状態にしています。

チャンスは逃さない。
フリラーンスをやっていると、何度もチャンスが巡ってきますが、これを掴むかどうかで長期的に成長できるかどうかが変わってきます。
例えば、バックエンドしかやったことがない状態で案件に入り、Reactなどのフロントエンドもやってみないか誘われたりすることは
よくありますが、自身の要望とマッチするならば、臆せずにチャレンジする方が良いです。


## 顧客が期待していること
時短、時間の節約、早期戦力化
顧客がフリーランスに期待することは、早期の戦力化です。
顧客の本音を言えば正社員を採用したいですが、必要なレベルの人材を短期で揃えることは難しいため、補充の意味でフリーランスと契約することが多いです。
つまり、キャッチアップに何ヶ月もかかるようでは期待を満たせません。
簡単なタスクをこなしながら１ヶ月くらいで普通に開発ができるようになる、くらいが期待値だと思います。

# 仕事の取りかた
エージェントが簡単
エージェントが用意するExcel表は使う必要はない

フリーランスの商品は自分自身なので、自分自身を売り込める必要がある
SESは担当営業が売り込んでくれるが、フリーランスは自分で営業する気持ちで

顧客もわざわざ中小のエージェントを使うメリットがないため、エージェントはおとなしく大手を使った方が良い。
大手エージェントは広告力があるので、エンジニアを集めやすいため、顧客が集まる。

最短１週間くらいで来まる。

エージェント経由×１
Twitter×１
Indeed×１
その他サイト×２


フレームワークが使えるとか、言語の理解などは最低限の話。

チャネルはいくらでも多くて良い
Twitterやその他SNS
Zennなどの技術記事、Githubなどあらゆるチャネルが仕事に繋がる可能性があるので
チャネルは多い方が良い。

受託開発は受けない方が良い
受託開発は、個人ではリスクが大きすぎるので、少なくとも初めのうちは受けるべきではないです。
瑕疵担保責任という数年に渡るバグ対応が求められる可能性があり、個人で負えるリスクではないです。
あまりビジネスっぽくはないですが、基本的には準委任の月単価、時間単価で働く方が確実です。

# フリーランスとしての最低ラインは？
私がバックエンドメインなので、バックエンドの場合を説明。

Git/Githubの基本的な機能(clone/push/pull/fetch/mergeなど)がCLIで使用でき、各機能の内容も説明できること
チーム開発の経験があり、PullRequestを使用したレビューができ、ブランチ開発が問題なく行えること
Docker/DockerComposeがCLIで使用できること。Dockerfileやdocker-compose.ymlを新規で記述できればベストだが
最低限、既存ファイルを見て概ねの内容を理解できること
AWS等のクラウドサービスやReact等のモダンフロントとバックエンドの関係性を理解しており説明できること
言語は問わないが、その言語メジャーなバックエンドフレームワークの環境構築、ユーザー認証機能ありでCRUD機能を持ったAPIを作成できること
クラスの抽象化の概念を理解して、実装することができる
何らかのリレーショナルDBを構築した経験があり、ORMを使用しない生のSQLを書くことができる。ER図が書ける、Indexの意味を理解して実装できる、
ORMを使用した構築経験がある

ランサーズやクラウドワークスなどのクラウドソーシングは、単価が非常に低く、顧客もごく小規模の場合が多いため
フリーランスの仕事としてはオススメできない。

# １日のリズムを意識する
一般的に、早朝の方が生産性が高く、夜は生産性が低い。私は１日を４ブロックに分けて、それぞれのブロックで行うタスクを区別しています。
早朝：一番生産性が高い時間帯なので、頭をつかうタスクを中心に行う
午前：早朝の次に生産性が高いので、継続して頭を使うタスクを行う
午後：集中力も切れてくる時間帯なので、その時々にあわせてルーチンタスクをおりまぜながら行う
夜：最も生産性が低い時間帯なので、開発作業には向かないので、ドキュメンテーションやSlack対応などの低負荷のタスクを行う

# 税知識
概ね年間８００万以上の収入が見込めるなら、個人での開業よりは法人化した方が税メリットがあると言われています。
税理士の無料相談もあるので、活用した方がよいです。

契約書は隅々まで読みましょう。
しれっと重要かつこちらに不利益なことが書かれているケースがあります。
不安であれば、単発で弁護士さんに確認するのも良いでしょう。
瑕疵担保責任やその有効期間などの、何かあった場合の取り決めがよく問題になりますので、注意しましょう。

# 設備
椅子
腰痛対策
PC：好きなものを選べば良いと思いますが、個人的にはubuntuがコストパフォーマンス最強だと思っています。自分はアプライドでubuntuPCを購入しました。
ディスプレイ：個人的にはマルチディスプレイが必須です。特にメインディスプレイはワイドディスプレイを推奨します

# IT課金
基本的にエンジニアは経費があまりかからないため、せめてIT関連だけはガンガン課金して生産性を上げた方が良いです。

Copilot必須
ChatGPT必須
