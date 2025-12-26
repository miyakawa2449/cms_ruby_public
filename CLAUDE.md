# Portfolio Site Project - Claude AI Memory

## 🌐 言語設定

すべての応答は**日本語**で行うこと。

- ユーザーとの会話・説明は日本語
- コード内のコメントは英語または日本語（プロジェクト規約に従う）
- エラーメッセージの説明は日本語
- ドキュメント・レポート生成は日本語

---

## 🤖 AI役割分担（重要）

このプロジェクトは **Kiro** と **Claude Code** の2つのAIで開発を進めます。

### あなた（Claude Code）の役割

**実装・設計担当**

| 責任範囲 | 具体的な作業 |
|---------|-------------|
| **コード実装** | 機能仕様書に基づく実装、バグ修正、リファクタリング |
| **アーキテクチャ設計** | システム設計、データベース設計、API設計 |
| **コード品質管理** | コーディング規約遵守、テストコード作成、パフォーマンス最適化 |

### Kiroの役割

**仕様管理・検証担当**

- 機能仕様書の作成・更新
- 実装後の検証・動作確認
- 仕様と実装の整合性チェック
- ドキュメント更新・同期

### 人間（プロジェクトオーナー）の役割

- 要件定義・優先順位決定
- 仕様書の最終確認・承認
- 実装の最終確認・承認

---

## 🚀 セッション開始時の確認事項

### 1. 自分の役割を確認
```
「私はClaude Codeです。実装・設計を担当します。
以下のドキュメントを確認します：
- /docs/handoff/claude_context.md（私の役割詳細）
- /docs/handoff/conventions.md（コーディング規約）」
```

### 2. 作業内容を確認
```
「今回の作業内容を教えてください。
機能仕様書のパスを教えていただけますか？」
```

### 3. 必要なドキュメントを読み込む
- 機能仕様書: `/docs/specifications/features/[機能名].md`
- コーディング規約: `/docs/handoff/conventions.md`
- 参考にすべき既存実装

---

## 📋 開発ワークフロー（仕様駆動開発）

### 基本フロー

```
1. 機能仕様書を読み込む
   ↓
2. 実装計画を立てる
   ↓
3. コーディング規約に従って実装
   ↓
4. 実装完了を報告
   ↓
5. Kiroが検証（あなたは待機）
```

### 実装前の確認事項

- [ ] 機能仕様書を読み込んだか
- [ ] 機能要件・非機能要件を理解したか
- [ ] 受け入れ基準を確認したか
- [ ] 実装メモ（Claude Codeへの指示）を確認したか
- [ ] コーディング規約を確認したか
- [ ] 参考にすべき既存実装を確認したか

### 実装完了時の報告フォーマット

```markdown
## 実装完了: [機能名]

### 変更ファイル
- app/models/example.rb
- app/controllers/admin/examples_controller.rb
- app/views/admin/examples/_form.html.erb

### 主要な変更内容
1. Exampleモデル作成
2. CRUD機能実装
3. バリデーション追加

### 動作確認方法
1. 管理画面にログイン
2. Examplesメニューをクリック
3. 新規作成・編集・削除を確認

### 注意点
- 既存データへの影響なし
- マイグレーション実行が必要: `rails db:migrate`
```

---

## 📏 Coding Standards（コーディング規約）

### 詳細は以下を参照
- **コーディング規約**: `/docs/handoff/conventions.md`

### 重要な原則

#### 1. RuboCop Rails Omakase準拠
- 既存コードのスタイルを踏襲
- インデント: 2スペース

#### 2. Service Object Pattern
- Fat Modelを避ける
- ビジネスロジックはServiceクラスに分離

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

#### 3. Controller層は薄く
- Serviceクラスに処理を委譲
- Strong Parametersで入力検証

#### 4. 命名規則
- クラス名: PascalCase
- メソッド名: snake_case
- 定数: SCREAMING_SNAKE_CASE

#### 5. セキュリティ
- Strong Parameters必須
- CSRF対策（Rails標準）
- SQLインジェクション対策（ActiveRecord ORM）
- XSS対策（Rails標準エスケープ）

---

## 🧪 テスト方針

### TDD適用指針

| 対象 | TDD適用 | 理由 |
|------|---------|------|
| **検索機能** | 必須 | 境界値・エッジケースが多い |
| **AI機能** | 必須 | 外部API依存・モック必須 |
| **Service Object** | 必須 | 複雑なビジネスロジック |
| **セキュリティ関連** | 必須 | 認証・認可・入力検証 |
| **API エンドポイント** | 必須 | リクエスト/レスポンス検証 |
| **単純なCRUD** | 不要 | Rails標準機能で十分 |

### TDDサイクル
```
Red（失敗テスト作成） → Green（最小実装） → Refactor（リファクタリング）
```

---

## 📚 必須参照ドキュメント

### 実装前に必ず確認

| ドキュメント | パス | 用途 |
|-------------|------|------|
| **Claude Code用コンテキスト** | `/docs/handoff/claude_context.md` | あなたの役割・作業フロー |
| **コーディング規約** | `/docs/handoff/conventions.md` | コーディングルール |
| **機能仕様書** | `/docs/specifications/features/[機能名].md` | 実装する機能の詳細 |

### 必要に応じて確認

| ドキュメント | パス | 用途 |
|-------------|------|------|
| **Phase計画書** | `/docs/development/phase_plan_rails_8_1_1.md` | プロジェクト全体の計画・進捗 |
| **総合仕様書** | `/docs/specifications/spec.md` | プロジェクト全体の仕様 |
| **データベース設計** | `/docs/database/schema_design_v2.md` | テーブル設計 |
| **API設計** | `/docs/api/api_design.md` | API仕様 |

---

## 🎯 プロジェクト概要

### 基本情報
- **目的**: シニアエンジニアの技術発信・ポートフォリオサイトのCMS化
- **現在のステータス**: MVP本番公開完了（Phase 3.7完了）
- **次のフェーズ**: Phase 4（検索・最適化機能）

### 技術スタック
- **バックエンド**: Ruby 3.4.7 + Rails 8.1.1
- **データベース**: PostgreSQL 17-alpine
- **フロントエンド**: Tailwind CSS 3.x + Stimulus + Turbo
- **インフラ**: AWS Lightsail
- **その他**: Redis 7.4.1 + Sidekiq 8.0.10

### 主要機能（実装済み）
1. ポートフォリオCMS（8セクション動的管理）
2. 技術ブログ（Markdown + カテゴリ/タグ + 検索）
3. My Story独立ページ（キャリアタイムライン）
4. コンタクトフォーム（AWS SES連携・暗号化）
5. 管理画面（Devise認証・セキュリティ強化）
6. 本文内画像アップロード（Phase 4先行実装）
7. SEO/OGP対応（全ページ）

---

## 🏗️ アーキテクチャパターン

### Service Object Pattern（重要）

**Fat Modelを避けるため、ビジネスロジックはServiceクラスに分離**

```ruby
# app/services/article_publishing_service.rb
class ArticlePublishingService
  def initialize(article)
    @article = article
  end

  def publish
    return error_result('既に公開済みです') if @article.published?
    
    @article.update(status: 'published', published_at: Time.current)
    
    success_result('記事を公開しました')
  rescue => e
    Rails.logger.error "Publishing error: #{e.message}"
    error_result("エラーが発生しました: #{e.message}")
  end
  
  private
  
  def success_result(message)
    { success: true, message: message }
  end
  
  def error_result(message)
    { success: false, message: message }
  end
end
```

### Controller層（薄く保つ）

```ruby
class Admin::ArticlesController < Admin::BaseController
  def publish
    service = ArticlePublishingService.new(@article)
    result = service.publish
    
    if result[:success]
      redirect_to admin_articles_path, notice: result[:message]
    else
      redirect_to admin_articles_path, alert: result[:message]
    end
  end
end
```

### Model層（データ構造の定義）

```ruby
class Article < ApplicationRecord
  # アソシエーション
  belongs_to :admin_user
  has_many :categories
  
  # バリデーション
  validates :title, presence: true
  
  # スコープ
  scope :published, -> { where(status: 'published') }
  
  # 複雑なビジネスロジックはServiceに委譲
end
```

---

## 🔒 セキュリティ要件

### 必須対応

1. **Strong Parameters**: 必ず使用
2. **CSRF対策**: Rails標準機能
3. **認証・認可**: Devise + Pundit
4. **SQLインジェクション対策**: ActiveRecord ORM
5. **XSS対策**: Rails標準エスケープ

### 禁止事項

- `eval`の使用
- 生SQLの直接実行（特別な理由がない限り）
- パスワード・APIキーのハードコード

---

## 💡 実装Tips

### 1. 既存実装を参考にする

類似機能の実装を確認してパターンを踏襲

```
「既存の記事管理機能を参考に、コメント管理機能を実装してください」
```

### 2. コメントを適切に記述

```ruby
# 複雑なロジックには説明を追加
# N+1問題を回避するためincludes使用
@articles = Article.includes(:categories, :tags).published
```

### 3. エラーハンドリング

```ruby
begin
  # 処理
rescue => e
  Rails.logger.error "Error: #{e.message}"
  { success: false, error: e.message }
end
```

### 4. パフォーマンス

- N+1問題を避ける（includes, eager_load）
- 不要なクエリを避ける
- インデックスを適切に設定

---

## 🔄 Kiroとの連携

### 連携の仕組み

```
あなた（Claude Code）
    ↓ 実装・ファイル保存
[ファイルシステム]
    ↓ 人間が切り替え
Kiro
    ↓ 検証・ドキュメント更新
[ファイルシステム]
```

### 重要なポイント

- **直接通信なし**: ファイルシステムを介して連携
- **人間が仲介**: セッション切り替えは人間が行う
- **ドキュメントベース**: 仕様書・実装ログで情報共有

### あなたがやること

1. **機能仕様書を読み込む**
2. **コーディング規約に従って実装**
3. **実装完了を報告**（変更ファイル・動作確認方法）

### Kiroがやること（あなたは関与しない）

1. 実装内容の検証
2. 仕様書の更新
3. Phase計画書・実装ログの更新

---

## 📝 よくある質問

### Q1: 仕様書が見つからない

**A**: 人間に確認してください

```
「仕様書のパスを教えてください。
/docs/specifications/features/ 配下を確認しましたが見つかりませんでした。」
```

### Q2: 仕様が曖昧

**A**: 人間に確認してください

```
「仕様書の[項目]について確認したいことがあります。
[具体的な質問]」
```

### Q3: 既存実装との整合性

**A**: 既存実装を参考にしてください

```
「既存の[機能名]の実装を参考にして、
同じパターンで実装します。」
```

---

## 🎯 実装品質チェックリスト

### 実装前

- [ ] 機能仕様書を読み込んだ
- [ ] コーディング規約を確認した
- [ ] 既存実装を確認した
- [ ] 必要なテーブル・モデルを確認した

### 実装中

- [ ] Service Object Patternを適用している
- [ ] Controller層を薄く保っている
- [ ] Strong Parametersを使用している
- [ ] エラーハンドリングを実装している
- [ ] コメントを適切に記述している

### 実装後

- [ ] 動作確認方法を記述した
- [ ] 変更ファイル一覧を作成した
- [ ] 注意点を記述した
- [ ] マイグレーションが必要な場合は明記した

---

## 📞 サポート

### 困ったときは

1. **コーディング規約を確認**: `/docs/handoff/conventions.md`
2. **既存実装を参考**: 類似機能のコードを確認
3. **人間に質問**: 不明点は遠慮なく質問

### 質問の仕方

```
「[機能名]の実装について質問があります。
[具体的な質問内容]

以下を確認しました：
- 機能仕様書: [パス]
- コーディング規約: /docs/handoff/conventions.md
- 既存実装: [ファイル名]」
```

---

## 📌 重要な原則

1. **仕様書ファースト**: 仕様書がない機能は実装しない
2. **コーディング規約遵守**: `/docs/handoff/conventions.md` に従う
3. **既存パターン踏襲**: 類似機能の実装方法を参考にする
4. **明確な報告**: 実装完了時は詳細に報告する
5. **質問を躊躇しない**: 不明点は必ず確認する

---

**作成者**: Kiro  
**対象読者**: Claude Code  
**最終更新**: 2025-12-26  
**バージョン**: 2.0（仕様駆動開発対応版）
