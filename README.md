# Portfolio Site CMS - Rails 8 ポートフォリオサイト

[![Rails](https://img.shields.io/badge/Rails-8.1.1-red.svg)](https://rubyonrails.org/)
[![Ruby](https://img.shields.io/badge/Ruby-3.4.7-red.svg)](https://www.ruby-lang.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-17-blue.svg)](https://www.postgresql.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 📌 このリポジトリについて

このリポジトリは、**既存プロジェクトへのKiro導入実験**のデモプロジェクトです。

開発中のポートフォリオサイトに対して、AWS提供の仕様駆動開発IDE「Kiro」を後から導入し、Claude Codeと併用した開発プロセスを検証するために切り出したものです。

**関連記事**: [Kiro + Claude Code 併用による仕様駆動開発、実験レポート](https://qiita.com/miyakawa2449@github/items/975176fed3b286cb3efa)

> ⚠️ **注意**: このリポジトリは実験記録用であり、これ以上の開発は進みません。実際の開発は別リポジトリで継続しています。

---

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

## Kiro導入実験について

このプロジェクトでは、以下のような開発プロセスを実験しました：

### 導入プロセス
1. **既存仕様書の読み込み**: Claude Code単体で進めていた仕様駆動開発の成果物をKiroに読み込ませる
2. **適応的な仕様書生成**: 既存の仕様書体系を壊さず、追加機能ごとに `.kiro/specs/` 配下に仕様書を生成
3. **役割分担の確立**: Kiroが仕様策定、Claude Codeが実装、人間が承認者として最小限の介入

### 実験結果
- ✅ タスク順守は高いレベルで実現
- ✅ 既存プロジェクトへの後付け導入も可能
- ⚠️ 人間の監督（承認者としての役割）は依然として必要

詳細は[Qiita記事](https://qiita.com/miyakawa2449@github/items/975176fed3b286cb3efa)をご覧ください。

## 注意事項

このリポジトリは**Qiita記事用のデモプロジェクト**として公開しています。

- 本番環境で使用する場合は、適切な環境変数の設定が必要です
- `config/credentials.yml.enc` は暗号化されており、`master.key` なしでは復号できません
- デプロイスクリプトや本番環境固有の設定は含まれていません
- **このリポジトリでの開発は終了しています**（実験記録用のスナップショット）

## ライセンス

MIT License

## 開発者

**Tsuyoshi Miyakawa**
Senior Software Engineer
