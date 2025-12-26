# Portfolio Site CMS - Rails 8 ポートフォリオサイト

[![Rails](https://img.shields.io/badge/Rails-8.1.1-red.svg)](https://rubyonrails.org/)
[![Ruby](https://img.shields.io/badge/Ruby-3.4.7-red.svg)](https://www.ruby-lang.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-17-blue.svg)](https://www.postgresql.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Rails 8.1 + Tailwind CSS で構築したポートフォリオ・ブログCMSの参考実装です。

## 主要機能

- **ポートフォリオCMS**: 8セクション構成の縦スクロール型サイト
- **技術ブログ**: Markdown対応 + カテゴリ/タグ管理
- **My Story**: キャリアタイムライン独立ページ
- **コンタクトフォーム**: メール送信対応
- **管理画面**: Devise認証・CRUD管理

## 技術スタック

### バックエンド
| 技術 | バージョン | 用途 |
|------|-----------|------|
| Ruby | 3.4.7 | 言語 |
| Rails | 8.1.1 | フレームワーク |
| PostgreSQL | 17-alpine | データベース |
| Solid Queue | - | バックグラウンドジョブ |

### フロントエンド
| 技術 | 用途 |
|------|------|
| Tailwind CSS | スタイリング |
| Stimulus | JavaScript |
| Turbo | SPA風画面遷移 |
| Propshaft | アセット管理 |

### 認証・セキュリティ
- **認証**: Devise
- **セキュリティ**: Rack::Attack、Brakeman

## セットアップ

### 前提条件
- Ruby 3.4.7
- Node.js 18+
- PostgreSQL 17+

### 環境変数の設定
```bash
# .env.example を参考に .env を作成
cp .env.example .env
# 必要な値を設定
```

### Docker開発環境（推奨）
```bash
# 環境構築・起動
docker-compose up -d

# データベースセットアップ
docker-compose exec web rails db:create db:migrate db:seed

# Railsサーバー起動
docker-compose exec -d web bundle exec rails server -b 0.0.0.0
```

### ローカル開発環境
```bash
# 依存関係インストール
bundle install

# データベースセットアップ
rails db:create db:migrate db:seed

# 開発サーバー起動
rails server
```

### 管理画面アクセス（開発環境）
```
URL: http://localhost:3000/admin
Email: admin@portfolio.dev
Password: password123
```

## テスト実行

```bash
# RSpecテストスイート
bundle exec rspec

# セキュリティ監査
bundle exec brakeman

# コード品質チェック
bundle exec rubocop
```

## プロジェクト構成

```
app/
├── controllers/
│   ├── admin/          # 管理画面コントローラー
│   ├── api/            # API コントローラー
│   └── ...             # フロントコントローラー
├── models/             # Active Record モデル
├── services/           # サービスクラス（ビジネスロジック）
├── views/
│   ├── admin/          # 管理画面ビュー
│   ├── portfolio/      # ポートフォリオビュー
│   └── blog/           # ブログビュー
└── ...
```

## アーキテクチャ

### Service Object Pattern
Fat Model を避け、ビジネスロジックは Service クラスに分離しています。

```ruby
# app/services/article_publishing_service.rb
class ArticlePublishingService
  def initialize(article)
    @article = article
  end

  def publish
    # ビジネスロジック
  end
end
```

## セキュリティ対策

- **認証**: Devise + セッション管理
- **セキュリティヘッダー**: Rack::Attack + CSP
- **入力検証**: Strong Parameters + バリデーション
- **SQLインジェクション対策**: ActiveRecord ORM
- **XSS対策**: Rails標準エスケープ
- **静的解析**: Brakeman（脆弱性検出）
- **依存関係監査**: bundler-audit

## 注意事項

このリポジトリは参考実装として公開しています。

- 本番環境で使用する場合は、適切な環境変数の設定が必要です
- `config/credentials.yml.enc` は暗号化されており、`master.key` なしでは復号できません
- デプロイスクリプトや本番環境固有の設定は含まれていません

## ライセンス

MIT License

## 開発者

**Tsuyoshi Miyakawa**
Senior Software Engineer
