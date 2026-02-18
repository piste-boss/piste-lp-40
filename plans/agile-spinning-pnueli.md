# フェーズ2 実装プラン

## Context

piste-lp-40のLPは、Clarityデータ分析によりフェーズ1.5まで改善を実施済み。中盤定着率は大幅改善したが、以下の課題が残存：

- **5-10%地点の「崖」**: 5%到達者の約65%が10%前に離脱（目標: 30%以下）
- **CTA行動率4.7%で停滞**: 目標8%に対して半分
- **年齢・健康への不安が最大の行動障壁**: FAQタップが上位独占

## 実装施策（4つ）

---

### 施策B-1: CTAマイクロコピー改善（最軽量・即効性）

**対象ファイル**: `index.html`(L32-34, L54-60, L250-252, L501-503), `index.css`

**HTML変更**:

1. **フローティングCTA**（index.html:32-34）:
   - 現在: `<a>` タグ内にテキストのみ
   - 変更後: `<a>` 内に `<span class="cta-main-text">` と `<span class="cta-microcopy">` を追加
   ```html
   <a href="..." class="floating-cta" target="_blank">
     <span class="cta-main-text">【無料】体験を予約する</span>
     <span class="cta-microcopy">同世代の方と一緒 ・ しつこい勧誘なし</span>
   </a>
   ```

2. **ヒーローCTA**（index.html:54-60）:
   - ボタンテキストはそのまま
   - 既存の `hero__social-proof`（L57-59）のテキストを変更:
   ```
   「同世代の方と一緒」「しつこい勧誘は一切なし」| 先月の体験予約: 13名
   ```

3. **キャンペーンセクションCTA**（index.html:250-252）, **最終CTA**（index.html:501-503）:
   - ボタン下にマイクロコピー `<p>` 要素を追加

**CSS変更**:
- `.floating-cta` を `display: flex; flex-direction: column;` に変更
- `.cta-main-text` と `.cta-microcopy` のスタイル追加
- `.cta-microcopy`: 小さめフォント、薄い色

---

### 施策B-2: FAQ内インラインCTA

**対象ファイル**: `index.html`(L119-151, L429-477), `index.css`

**HTML変更**:
- 早期FAQ 3問 + 詳細FAQ 4問 = 計7箇所の `.faq__answer` 末尾にインラインCTA追加
- FAQ内容に応じたバリエーション:
  - 食事制限FAQ: `→ 栄養指導も含めた無料体験はこちら`
  - 体力FAQ: `→ 運動未経験の方も安心。まずは無料体験を`
  - 腰痛FAQ: `→ メンテナンス感覚の無料体験を予約する`
  - 年齢FAQ: `→ 同世代の方と一緒に、まずは無料体験を`
  - 健康FAQ: `→ 医師連携の安心環境を、まずは体験してみませんか？`
  - その他: `→ まずは雰囲気を見てみませんか？【無料体験を予約】`

**CSS変更**:
- `.faq__inline-cta`: FAQ回答内に自然に馴染むリンクスタイル（矢印アイコン付き、オレンジ系色）

---

### 施策A-1: 信頼バッジ → 同世代の声カルーセルへ置換

**対象ファイル**: `index.html`(L71-81), `index.css`

**重要**: `section.trust-badges` は `div.hero__container` の中（`section.hero` 内部）にネストされている。カルーセルも同じ位置（hero内部）に配置する。

**HTML変更**:
- `section.trust-badges`（index.html:71-81）を削除し、同じ位置に以下を挿入:
  ```html
  <div class="voice-carousel" aria-label="利用者の声">
    <div class="voice-carousel__track">
      <div class="voice-slide voice-slide--active">
        <p class="voice-slide__text">「58歳、膝の手術後でも<br>安心して通えています」</p>
        <p class="voice-slide__author">58歳 男性 会社員</p>
      </div>
      <div class="voice-slide">
        <p class="voice-slide__text">「高血圧を指摘されていましたが、<br>医師連携で安心です」</p>
        <p class="voice-slide__author">52歳 女性 主婦</p>
      </div>
      <div class="voice-slide">
        <p class="voice-slide__text">「半年続いています。<br>整体感覚で通えるのが良い」</p>
        <p class="voice-slide__author">61歳 男性 自営業</p>
      </div>
    </div>
    <div class="voice-dots">
      <button class="voice-dot voice-dot--active" aria-label="スライド1"></button>
      <button class="voice-dot" aria-label="スライド2"></button>
      <button class="voice-dot" aria-label="スライド3"></button>
    </div>
    <p class="voice-scroll-hint">↓ 同世代の方の体験を見る</p>
  </div>
  ```

**CSS変更**:
- 旧スタイル削除: `.trust-badges`, `.trust-badge-item`, `.trust-badge-icon`, `.trust-badge-main`, `.trust-badge-sub` とそのレスポンシブ
- 新規スタイル: `.voice-carousel`, `.voice-carousel__track`, `.voice-slide`, `.voice-dots`, `.voice-dot`, `.voice-scroll-hint`
- スライドはフェードイン/アウトで切り替え（transform不使用でシンプルに）

**JS変更**（index.html末尾のscriptタグ内に追記）:
- 3秒間隔の自動スライド
- ドットクリックでスライド切り替え
- **タッチ開始時に自動スライドを一時停止**（3秒後に再開）
- アクセシビリティ: `aria-live="polite"` をtrack要素に追加

---

### 施策A-2: 30秒診断インタラクティブ要素

**対象ファイル**: `index.html`, `index.css`

**前提**: スクロール先セクションにid属性を追加する必要あり
- `section.intro` → `id="intro"` を追加
- `section.reasons` → `id="reasons"` を追加
- `section.price` → 既に `id="price"` あり（変更不要）

**HTML変更**:
- カルーセルの下、`div.scroll-hint`（index.html:87-89）の前に配置:
  ```html
  <div class="quick-diagnosis">
    <p class="diagnosis-title">あなたに合った運動を10秒で診断</p>
    <p class="diagnosis-question">Q. 一番気になることは？</p>
    <div class="diagnosis-buttons">
      <button class="diagnosis-btn" data-target="#intro">体力に自信がない</button>
      <button class="diagnosis-btn" data-target="#reasons">膝や腰に不安がある</button>
      <button class="diagnosis-btn" data-target="#price">続けられるか心配</button>
    </div>
  </div>
  ```

**CSS変更**:
- `.quick-diagnosis`, `.diagnosis-title`, `.diagnosis-question`, `.diagnosis-btn` のスタイル

**JS変更**:
- `data-target` 属性の値を使って `scrollIntoView({ behavior: 'smooth' })` を実行

---

## 実装順序

1. **施策B-1**: CTAマイクロコピー改善
2. **施策B-2**: FAQ内インラインCTA
3. **施策A-1**: カルーセル実装（旧trust-badges削除 + 新カルーセル挿入）
4. **施策A-2**: 30秒診断 + セクションへのid追加

## 変更対象ファイル

| ファイル | 変更内容 |
|:---|:---|
| `index.html` | CTAマイクロコピー、FAQ内CTA、カルーセルHTML、診断UI、セクションid追加 |
| `index.css` | マイクロコピースタイル、インラインCTA、カルーセルスタイル、診断スタイル、旧trust-badges CSS削除 |

## 検証方法

1. ブラウザでindex.htmlを開いてモバイルビュー（375px幅）で確認
2. カルーセルが3秒間隔で自動スライドすることを確認
3. ドットインジケーターのクリックでスライド切り替えを確認
4. カルーセルをタッチした際に自動スライドが一時停止することを確認
5. 30秒診断の各ボタンクリックで正しいセクションにスムーズスクロールすることを確認
6. すべてのCTAにマイクロコピーが表示されていることを確認
7. FAQ項目を展開した際に、回答末尾にインラインCTAが表示されることを確認
8. すべてのCTAリンクが `https://piste-reserve.netlify.app/lp` に遷移することを確認
9. フローティングCTAが2行表示（メインテキスト + マイクロコピー）になっていることを確認
