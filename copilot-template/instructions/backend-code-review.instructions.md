---
applyTo: "**/*.{py},**/*.{ts}"
description: "バックエンド(FastAPI/NestJsなど)のコードレビューの観点やルールをまとめている。"
---

# バックエンドコードレビューガイドライン

## 前提事項

- 技術スタック
  - **言語**: Python, TypeScript
  - **フレームワーク**: FastAPI, NestJs
  - **ORM**: SQLAlchemy, TypeORM
  - **DB**: PostgreSQL
- アーキテクチャレイヤー
  - **レイヤー構成**: views, adapter, application, domain
    - 同一レイヤーへの参照を許可
    - views > applicationへの参照を許可
    - views > domainへの参照を許可
    - adapter > applicationへの参照を許可
    - adapter > domainへの参照を許可
    - application > domainへの参照を許可
  - **adapter**
    - WebAPIとしてHTTPリクエストを受け取り、内部のビジネスロジックであるapplication層に処理を委譲する役割を担う
  - **application**
    - アダプター層からデータを受け取り、ドメイン層のビジネスロジックを呼び出す役割を担う
  - **domain**
    - ビジネス領域における純粋な業務ルールやデータ構造を定義する層で、アーキテクチャの中心となる。フレームワークやDB事情などに絶対に依存してはいけない
- タイムゾーン
  - アプリ内では常にUTCを使用すること
  - DBに保存する日時はUTCで保存すること
  - コンソールログやファイル名など、ユーザーが直接目にする日時は設定されたタイムゾーンで表示すること
- API設計
  - **画面特化API(BFF)**: `/views/{page-name}/{ids}`形式とし、ページ単位でAPIを設計する
  - **REST API**: `/{resource-name}/{ids}`形式とし、集約ルート単位でAPIを設計する
- **Global Exception Filter**: アプリケーション全体で一貫した例外処理を実装すること
- データベース
  - DB依存の機能利用を禁止する
  - DBのカラム名はスネークケースで命名すること
- 命名規則
  - ファイル名はコンポーネント識別をサフィックスで行うこと
    - 例: `artist_controller.py`, `artist_service.py`, `artist_repository.py`
    - 例: `artist.controller.ts`, `artist.service.ts`, `artist.repository.ts`
  - クラス名はファイル名をアッパーキャメルケースに変換したものとすること
    - 例: `ArtistController`, `ArtistService`, `ArtistRepository`
- レビュー実施時はレビューテンプレートを使用すること
  - 重要度
    - **High**: 修正必須
    - **Medium**: 特別な事情がない限り修正推奨
    - **Low**: 任意だが、可能なら修正推奨
    - **Suggestion**: 不具合等ではないが、コード品質向上のための提案

## レビュー指摘テンプレート

```md
## <!-- 指摘内容の要約 -->

- **重要度**: [ 🚨High / ⚠️Medium / ℹ️Low / 💡Suggestion ]
- **分類**: [ ❄️Bug / 🐧Refactoring / ⭐Performance / 🔒Security / 📝Documentation / 🎀Design / ☕️Other ]

**詳細**

<!-- 指摘内容の詳細を記述する -->

**補足**

<!-- 補足事項があれば記述する。また同様の指摘内容が他の箇所にも存在する場合は、そのファイルパスを列挙する。 -->
```

## レビュー観点

- アーキテクチャ違反がないか
  - 許可されていないレイヤー参照方向が無いか
  - 各レイヤー、コンポーネントは適切な責務を担っているか
  - contexts/{resource}/にはフレームワークやDB依存のコードが混入していないか
- API設計ルールに従っているか
- DB依存の機能利用が無いか
- 命名規則に従っているか
- 例外処理はGlobalExceptionFilterで一元管理されているか
- タイムゾーンはアプリ内部ではUTCを使用し、ユーザーが直接目にする日時は設定されたタイムゾーンで表示されているか
- REST APIの原則に従っているか
- 自明な処理を除き、適切なコメントが記述されているか