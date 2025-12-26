# 記事画像キャプション機能 仕様書

## 📅 作成日・更新日
- **作成日**: 2024-12-26
- **最終更新**: 2025-12-26
- **ステータス**: ✅ 実装完了

---

## 🎯 概要

### 目的
記事内に挿入する画像に対して、alt属性だけでなくキャプション（figcaption）を追加できるようにする。
これにより、画像の詳細な説明を視覚的に表示し、読者の理解を深める。

### ユーザーストーリー
- 記事執筆者として、画像にキャプションを付けたい、なぜなら画像の詳細な説明を読者に提供したいから
- 記事執筆者として、alt属性とキャプションを使い分けたい、なぜならSEO/アクセシビリティとユーザー体験の両方を最適化したいから

---

## ✅ 要件

### 機能要件
- [ ] 画像アップロード時にキャプション入力フィールドを追加
- [ ] キャプションが入力された場合、`<figure>`と`<figcaption>`タグを使用したHTMLを生成
- [ ] キャプションが空の場合、従来通りシンプルなMarkdown形式（`![alt](url)`）で挿入
- [ ] 生成されたHTMLは記事本文（Markdown）に挿入される
- [ ] フロントエンド表示時に適切なスタイリングが適用される

### 非機能要件
- **パフォーマンス**: 画像アップロード処理に影響を与えない
- **セキュリティ**: キャプションテキストのXSS対策（HTMLエスケープ）
- **アクセシビリティ**: alt属性とfigcaptionの適切な使い分け
- **SEO**: 画像検索最適化のためのalt属性とキャプションの併用

---

## 🖼️ 画面仕様

### UI/UX詳細

#### 管理画面（記事編集フォーム）
現在の画像アップロードセクションに以下を追加：

```
📷 本文内画像アップロード
┌─────────────────────────────────────┐
│ 画像ファイル: [ファイル選択ボタン]    │
│                                     │
│ 代替テキスト（alt属性）              │
│ [画像の説明を入力してください]       │
│ ℹ️ SEO・アクセシビリティのため       │
│                                     │
│ キャプション（任意）                 │
│ [画像のキャプションを入力...]        │
│ ℹ️ 画像の下に表示される説明文です。  │
│    空欄の場合はシンプルなMarkdown    │
│    形式で挿入されます。              │
│                                     │
│ [画像をアップロードして本文に挿入]   │
└─────────────────────────────────────┘
```

#### フロントエンド表示

キャプション付き画像の表示例：

```html
<figure class="article-image">
  <img src="http://example.com/foo.jpg" alt="画像説明" />
  <figcaption>キャプション（詳細説明）</figcaption>
</figure>
```

スタイリング要件：
- 画像は中央寄せ
- キャプションは画像の下に表示、グレーの小さめのフォント
- レスポンシブ対応（モバイルでも適切に表示）

---

## 🗄️ データ仕様

### 使用するモデル
- **Article**: 既存モデル（変更なし）
- **ActiveStorage::Attachment**: 画像ファイル管理（変更なし）

### データフロー
```
1. ユーザーが画像ファイル、alt属性、キャプションを入力
2. JavaScriptがFormDataを作成してPOST
3. Railsコントローラーが画像をActive Storageに保存
4. キャプションの有無で生成するHTMLを分岐
   - キャプションあり: <figure>タグのHTML
   - キャプションなし: ![alt](url) のMarkdown
5. 生成されたコードを記事本文に挿入
```

---

## 🔌 API仕様

### エンドポイント
```
POST /admin-secure-panel-miyakawa2449/articles/:article_id/images
```

### リクエストパラメータ（変更後）
```javascript
FormData {
  image: File,           // 画像ファイル
  alt_text: String,      // alt属性（既存）
  caption: String        // キャプション（新規・任意）
}
```

### レスポンス例（キャプションあり）
```json
{
  "success": true,
  "markdown": "<figure class=\"article-image\">\n  <img src=\"http://localhost:3000/rails/active_storage/blobs/.../image.jpg\" alt=\"画像説明\" />\n  <figcaption>キャプション文</figcaption>\n</figure>",
  "url": "http://localhost:3000/rails/active_storage/blobs/.../image.jpg",
  "alt_text": "画像説明",
  "caption": "キャプション文",
  "filename": "image.jpg"
}
```

### レスポンス例（キャプションなし）
```json
{
  "success": true,
  "markdown": "![画像説明](http://localhost:3000/rails/active_storage/blobs/.../image.jpg)",
  "url": "http://localhost:3000/rails/active_storage/blobs/.../image.jpg",
  "alt_text": "画像説明",
  "caption": null,
  "filename": "image.jpg"
}
```

---

## 🧪 受け入れ基準

実装完了の判断基準：

- [x] 画像アップロードフォームにキャプション入力フィールドが追加されている
- [x] キャプションを入力して画像をアップロードすると、`<figure>`タグのHTMLが本文に挿入される
- [x] キャプションを空欄にして画像をアップロードすると、従来通りMarkdown形式で挿入される
- [x] 挿入されたHTMLのalt属性とキャプションが正しく設定されている
- [x] フロントエンドで画像とキャプションが適切にスタイリングされて表示される
- [x] キャプションテキストがHTMLエスケープされ、XSS攻撃を防げる
- [x] モバイル表示でも画像とキャプションが適切に表示される
- [x] 既存の画像アップロード機能（キャプションなし）が正常に動作する

---

## 💡 実装メモ

### 実装対象ファイル

1. **ビュー**: `app/views/admin/articles/_form.html.erb`
   - キャプション入力フィールドの追加
   - Stimulus targetの追加: `data-image-upload-target="caption"`

2. **JavaScript**: `app/javascript/controllers/image_upload_controller.js`
   - `caption` targetの追加
   - `uploadAndInsert`メソッドでcaptionをFormDataに追加
   - `resetForm`メソッドでcaptionフィールドもクリア

3. **コントローラー**: `app/controllers/admin/article_images_controller.rb`
   - `params[:caption]`を受け取る
   - キャプションの有無で生成するコードを分岐
   - HTMLエスケープ処理（`ERB::Util.html_escape`）

4. **スタイルシート**: `app/assets/stylesheets/application.tailwind.css` または専用CSS
   - `.article-image` クラスのスタイリング
   - `figcaption` のスタイリング

### 実装例（コントローラー）

```ruby
def create
  @article = Article.find(params[:article_id])
  
  if params[:image].present?
    @article.content_images.attach(params[:image])
    image = @article.content_images.last
    
    if image.present?
      image_url = url_for(image)
      alt_text = params[:alt_text].presence || image.filename.to_s
      caption = params[:caption].presence
      
      # キャプションの有無で生成するコードを分岐
      markdown = if caption
        # HTMLエスケープ
        escaped_alt = ERB::Util.html_escape(alt_text)
        escaped_caption = ERB::Util.html_escape(caption)
        
        <<~HTML.strip
          <figure class="article-image">
            <img src="#{image_url}" alt="#{escaped_alt}" />
            <figcaption>#{escaped_caption}</figcaption>
          </figure>
        HTML
      else
        "![#{alt_text}](#{image_url})"
      end
      
      render json: {
        success: true,
        markdown: markdown,
        url: image_url,
        alt_text: alt_text,
        caption: caption,
        filename: image.filename.to_s
      }
    end
  end
end
```

### CSS実装例

```css
/* 記事内画像のスタイリング */
.article-image {
  margin: 2rem auto;
  text-align: center;
  max-width: 100%;
}

.article-image img {
  max-width: 100%;
  height: auto;
  border-radius: 0.5rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.article-image figcaption {
  margin-top: 0.75rem;
  font-size: 0.875rem;
  color: #6b7280;
  font-style: italic;
  line-height: 1.5;
}

/* レスポンシブ対応 */
@media (max-width: 640px) {
  .article-image {
    margin: 1.5rem auto;
  }
  
  .article-image figcaption {
    font-size: 0.8125rem;
  }
}
```

### セキュリティ考慮事項
- キャプションテキストは必ず`ERB::Util.html_escape`でエスケープ
- alt属性も同様にエスケープ
- 画像URLは`url_for`で生成されるため安全

### 技術的制約
- Markdownパーサー（Redcarpet）はHTMLタグをそのまま通すため、`<figure>`タグは問題なく表示される
- 既存のMarkdown記法との互換性を保つため、キャプションなしの場合は従来通りの動作を維持

---

## 📊 実装履歴

| 日付 | 担当 | 内容 |
|------|------|------|
| 2024-12-26 | Kiro | 初版作成（Phase5実装待ち） |
| 2025-12-26 | Claude Code | 実装完了（Phase4.2） |

---

## 🔗 関連ドキュメント

- Phase計画書: `/docs/development/phase_plan_rails_8_1_1.md`
- 総合仕様書: `/docs/specifications/spec.md`
- 既存実装: `app/javascript/controllers/image_upload_controller.js`
- 既存実装: `app/controllers/admin/article_images_controller.rb`

---

## 📝 補足

### alt属性とfigcaptionの使い分け

- **alt属性**: 画像が表示されない場合の代替テキスト（スクリーンリーダー、画像読み込み失敗時）
  - 簡潔に画像の内容を説明
  - SEO対策として重要
  
- **figcaption**: 画像の補足説明やキャプション
  - 視覚的に表示される説明文
  - より詳細な情報や文脈を提供
  - 画像の出典、撮影情報なども記載可能

### 実装優先度
Phase4での実装を予定。Phase4.1（基本画像アップロード）完了後に着手する。
