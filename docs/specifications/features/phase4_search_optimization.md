# ブログ検索・最適化機能 仕様書

## 📅 作成日・更新日
- **作成日**: 2024-12-26
- **最終更新**: 2024-12-26
- **ステータス**: 🔵 実装待ち（Phase4.3予定）

---

## 🎯 概要

### 目的
ブログ記事の検索機能を実装し、ユーザーが目的の記事を素早く見つけられるようにする。
また、基本的なキャッシュ戦略を導入してパフォーマンスを最適化する。

### ユーザーストーリー
- 読者として、キーワードで記事を検索したい、なぜなら興味のある記事を素早く見つけたいから
- 読者として、カテゴリやタグで記事を絞り込みたい、なぜなら特定のトピックの記事をまとめて読みたいから
- 読者として、検索結果が素早く表示されてほしい、なぜなら待ち時間なく快適に閲覧したいから

---

## ✅ 要件

### 機能要件
- [ ] **基本検索機能**
  - キーワード検索（記事タイトル・本文・抜粋）
  - 検索結果一覧表示
  - 検索キーワードのハイライト表示
  - 検索結果のページネーション
  
- [ ] **フィルタリング機能**
  - カテゴリによる絞り込み
  - タグによる絞り込み
  - 公開ステータスによる絞り込み（管理画面のみ）
  - 複数条件の組み合わせ
  
- [ ] **検索UI**
  - ブログ一覧ページに検索バー配置
  - フィルタUIの実装（ドロップダウン/チェックボックス）
  - 検索中のローディング表示
  - 検索結果件数の表示

- [ ] **パフォーマンス最適化**
  - 検索結果のキャッシュ（頻繁に検索されるクエリ）
  - カテゴリ/タグ一覧のキャッシュ
  - N+1クエリの解消
  - データベースインデックスの最適化

### 非機能要件
- **パフォーマンス**: 検索レスポンスタイム < 500ms
- **スケーラビリティ**: 1000記事以上でも快適に動作
- **ユーザビリティ**: 直感的な検索UI、モバイル対応
- **SEO**: 検索結果ページのSEO対応（メタタグ、URL構造）

---

## 🖼️ 画面仕様

### UI/UX詳細

#### ブログ一覧ページ（検索UI追加）

```
┌─────────────────────────────────────────────────┐
│ Blog                                            │
├─────────────────────────────────────────────────┤
│                                                 │
│ 🔍 [キーワードを入力...        ] [検索]        │
│                                                 │
│ カテゴリ: [すべて ▼]  タグ: [すべて ▼]        │
│                                                 │
│ 検索結果: 15件                                  │
│                                                 │
│ ┌─────────────────────────────────────────┐   │
│ │ 記事タイトル（キーワードがハイライト）    │   │
│ │ 2024-12-26 | カテゴリ名                  │   │
│ │ 記事の抜粋...キーワード...                │   │
│ │ #タグ1 #タグ2                            │   │
│ └─────────────────────────────────────────┘   │
│                                                 │
│ [1] 2 3 4 ... 10 次へ >                        │
└─────────────────────────────────────────────────┘
```

#### 検索結果ページ（/blog/search）

```
┌─────────────────────────────────────────────────┐
│ 検索結果: "Rails" - 15件                        │
├─────────────────────────────────────────────────┤
│                                                 │
│ 🔍 [Rails              ] [検索]                │
│                                                 │
│ フィルタ:                                       │
│ ☑ カテゴリ: プログラミング (10)                │
│ ☐ カテゴリ: インフラ (5)                       │
│                                                 │
│ ☑ タグ: Ruby (8)                               │
│ ☑ タグ: Rails (15)                             │
│                                                 │
│ ┌─────────────────────────────────────────┐   │
│ │ Railsで始めるWeb開発                     │   │
│ │ 2024-12-20 | プログラミング               │   │
│ │ Railsは強力なWebフレームワーク...        │   │
│ │ #Ruby #Rails #Web開発                    │   │
│ └─────────────────────────────────────────┘   │
│                                                 │
└─────────────────────────────────────────────────┘
```

### レスポンシブ対応

**モバイル表示**:
- 検索バーを全幅表示
- フィルタをアコーディオン形式で折りたたみ
- 検索結果をカード形式で縦並び

---

## 🗄️ データ仕様

### 使用するモデル
- **Article**: 既存モデル（検索対象）
- **Category**: カテゴリフィルタ
- **Tag**: タグフィルタ

### 検索対象フィールド
```ruby
# Articleモデルの検索対象
- title (タイトル) - 重み: 高
- excerpt (抜粋) - 重み: 中
- content (本文) - 重み: 低
- meta_description (メタ説明) - 重み: 中
```

### データベースインデックス追加

```ruby
# 検索パフォーマンス向上のためのインデックス
add_index :articles, :title
add_index :articles, :status
add_index :articles, :published_at
add_index :articles, [:status, :published_at]

# 全文検索用（PostgreSQL）
execute <<-SQL
  CREATE INDEX articles_search_idx ON articles 
  USING gin(to_tsvector('japanese', 
    coalesce(title, '') || ' ' || 
    coalesce(excerpt, '') || ' ' || 
    coalesce(content, '')
  ));
SQL
```

### データフロー

```
1. ユーザーが検索キーワード入力
2. フロントエンドがGETリクエスト送信
   GET /blog/search?q=keyword&category=1&tags[]=2
3. コントローラーがパラメータ解析
4. キャッシュ確認（ヒットすれば返却）
5. データベース検索実行
   - LIKE検索 or PostgreSQL全文検索
   - カテゴリ/タグでフィルタ
   - ページネーション適用
6. 検索結果をキャッシュに保存
7. ビューに検索結果を渡す
8. 検索キーワードをハイライト表示
```

---

## 🔌 API仕様

### エンドポイント

#### 検索API（公開）
```
GET /blog/search
GET /api/v1/articles/search
```

### リクエストパラメータ

```ruby
{
  q: String,              # 検索キーワード（任意）
  category_id: Integer,   # カテゴリID（任意）
  tag_ids: Array,         # タグID配列（任意）
  page: Integer,          # ページ番号（デフォルト: 1）
  per_page: Integer       # 1ページあたりの件数（デフォルト: 10）
}
```

### レスポンス例

```json
{
  "success": true,
  "data": {
    "articles": [
      {
        "id": 1,
        "title": "Railsで始めるWeb開発",
        "slug": "rails-web-development",
        "excerpt": "Railsは強力なWebフレームワーク...",
        "published_at": "2024-12-20T10:00:00Z",
        "thumbnail_url": "https://example.com/image.jpg",
        "categories": [
          {
            "id": 1,
            "name": "プログラミング",
            "slug": "programming"
          }
        ],
        "tags": [
          {
            "id": 1,
            "name": "Ruby"
          },
          {
            "id": 2,
            "name": "Rails"
          }
        ],
        "highlight": {
          "title": "<mark>Rails</mark>で始めるWeb開発",
          "excerpt": "<mark>Rails</mark>は強力なWebフレームワーク..."
        }
      }
    ],
    "meta": {
      "total_count": 15,
      "current_page": 1,
      "total_pages": 2,
      "per_page": 10,
      "query": "Rails",
      "filters": {
        "category_id": 1,
        "tag_ids": [1, 2]
      }
    }
  }
}
```

---

## 🧪 受け入れ基準

実装完了の判断基準：

### 検索機能
- [ ] キーワード検索が正常に動作する（タイトル・本文・抜粋から検索）
- [ ] 検索結果にキーワードがハイライト表示される
- [ ] 検索結果が正しくページネーションされる
- [ ] 検索結果件数が正しく表示される
- [ ] 検索キーワードが空の場合、全記事が表示される

### フィルタリング機能
- [ ] カテゴリで絞り込みができる
- [ ] タグで絞り込みができる
- [ ] カテゴリとタグの複数条件で絞り込みができる
- [ ] フィルタ適用後も検索キーワードが保持される

### UI/UX
- [ ] 検索バーが直感的に使える
- [ ] フィルタUIが使いやすい
- [ ] 検索中のローディング表示が適切
- [ ] モバイル表示が適切に動作する
- [ ] 検索結果ページのSEO対応（title、meta description）

### パフォーマンス
- [ ] 検索レスポンスタイムが500ms以内
- [ ] N+1クエリが発生していない
- [ ] 頻繁な検索クエリがキャッシュされている
- [ ] 1000記事以上でも快適に動作する

---

## 💡 実装メモ

### 実装対象ファイル

1. **コントローラー**: `app/controllers/blog_controller.rb`
   - `search` アクションの追加
   - パラメータ処理・バリデーション
   - キャッシュ処理

2. **モデル**: `app/models/article.rb`
   - `search` スコープの追加
   - 検索ロジックの実装
   - フィルタリングロジック

3. **ビュー**: `app/views/blog/search.html.erb`
   - 検索結果表示
   - フィルタUI
   - ページネーション

4. **ビュー**: `app/views/blog/index.html.erb`
   - 検索バーの追加
   - フィルタUIの追加

5. **JavaScript**: `app/javascript/controllers/search_controller.js`
   - 検索フォーム送信処理
   - フィルタ変更時の処理
   - ローディング表示

6. **ヘルパー**: `app/helpers/search_helper.rb`
   - キーワードハイライト処理
   - 検索結果フォーマット

### 実装アプローチ

#### Phase 4.3（基本検索）
**シンプルなLIKE検索を使用**:
```ruby
# app/models/article.rb
scope :search, ->(query) {
  return all if query.blank?
  
  where(
    "title ILIKE :q OR excerpt ILIKE :q OR content ILIKE :q",
    q: "%#{sanitize_sql_like(query)}%"
  )
}

scope :by_category, ->(category_id) {
  return all if category_id.blank?
  joins(:categories).where(categories: { id: category_id })
}

scope :by_tags, ->(tag_ids) {
  return all if tag_ids.blank?
  joins(:tags).where(tags: { id: tag_ids })
}
```

#### Phase 5（高度な検索）で実装予定
- PostgreSQL全文検索（pg_search gem）
- インクリメンタルサーチ（リアルタイム検索）
- 検索候補の表示
- 検索履歴機能

### キャッシュ戦略

```ruby
# app/controllers/blog_controller.rb
def search
  cache_key = "blog_search/#{params[:q]}/#{params[:category_id]}/#{params[:tag_ids]}/#{params[:page]}"
  
  @articles = Rails.cache.fetch(cache_key, expires_in: 1.hour) do
    Article.published
           .search(params[:q])
           .by_category(params[:category_id])
           .by_tags(params[:tag_ids])
           .order(published_at: :desc)
           .page(params[:page])
           .per(10)
  end
end
```

### パフォーマンス最適化

1. **Eager Loading**:
```ruby
Article.includes(:categories, :tags, thumbnail_image_attachment: :blob)
```

2. **データベースインデックス**:
```ruby
# db/migrate/XXXXXX_add_search_indexes.rb
class AddSearchIndexes < ActiveRecord::Migration[8.0]
  def change
    add_index :articles, :title
    add_index :articles, :status
    add_index :articles, [:status, :published_at]
  end
end
```

3. **カウンタキャッシュ**:
```ruby
# カテゴリ・タグの記事数をキャッシュ
add_column :categories, :articles_count, :integer, default: 0
add_column :tags, :articles_count, :integer, default: 0
```

### キーワードハイライト実装

```ruby
# app/helpers/search_helper.rb
module SearchHelper
  def highlight_keywords(text, query)
    return text if query.blank?
    
    # HTMLエスケープしてからハイライト
    escaped_text = ERB::Util.html_escape(text)
    keywords = query.split(/\s+/)
    
    keywords.each do |keyword|
      escaped_keyword = Regexp.escape(keyword)
      escaped_text = escaped_text.gsub(
        /#{escaped_keyword}/i,
        '<mark>\0</mark>'
      )
    end
    
    escaped_text.html_safe
  end
  
  def truncate_with_highlight(text, query, length: 200)
    # キーワード周辺のテキストを抽出
    if query.present? && text.include?(query)
      start_pos = [text.index(query) - 50, 0].max
      truncated = text[start_pos, length]
      "...#{highlight_keywords(truncated, query)}..."
    else
      truncate(text, length: length)
    end
  end
end
```

### SEO対応

```ruby
# app/controllers/blog_controller.rb
def search
  @query = params[:q]
  @articles = # ... 検索処理
  
  # SEOメタタグ設定
  set_meta_tags(
    title: "「#{@query}」の検索結果 - Blog",
    description: "「#{@query}」に関する記事#{@articles.total_count}件を表示しています。",
    noindex: true  # 検索結果ページはインデックスしない
  )
end
```

### ルーティング

```ruby
# config/routes.rb
get 'blog/search', to: 'blog#search', as: :blog_search

namespace :api do
  namespace :v1 do
    resources :articles do
      collection do
        get :search
      end
    end
  end
end
```

---

## 📊 実装履歴

| 日付 | 担当 | 内容 |
|------|------|------|
| 2024-12-26 | Kiro | 初版作成（Phase4.3実装待ち） |

---

## 🔗 関連ドキュメント

- Phase計画書: `/docs/development/phase_plan_rails_8_1_1.md`
- 総合仕様書: `/docs/specifications/spec.md`
- 既存実装: `app/controllers/blog_controller.rb`
- 既存実装: `app/models/article.rb`

---

## 📝 補足

### Phase 4 vs Phase 5の検索機能比較

| 機能 | Phase 4（基本検索） | Phase 5（高度な検索） |
|------|-------------------|---------------------|
| 検索方式 | LIKE検索 | PostgreSQL全文検索 |
| 検索速度 | 中程度 | 高速 |
| 検索精度 | 部分一致 | 形態素解析・スコアリング |
| リアルタイム検索 | ❌ | ✅ |
| 検索候補 | ❌ | ✅ |
| 検索履歴 | ❌ | ✅ |
| ファセット検索 | 基本的なフィルタ | 高度なファセット |

### 実装優先度
Phase 4.3での実装を予定。Phase 4.2（画像キャプション）完了後に着手する。

### 技術的制約
- PostgreSQL 16の機能を活用
- Rails 8.0.4のキャッシュ機能を使用
- 既存のkaminari gemでページネーション
- 日本語検索に対応（ILIKE使用）

### パフォーマンス目標
- 検索レスポンスタイム: < 500ms
- 同時検索リクエスト: 100req/s
- キャッシュヒット率: > 70%
- データベースクエリ数: < 5 queries/request
