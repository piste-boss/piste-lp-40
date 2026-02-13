# フェーズ1.5 実装プラン — 5-10%地点の離脱「崖」解消

## Context

ヒートマップ分析で5-10%地点に離脱の崖が発生していることが判明。ヒーロー直後に信頼構築要素がなく、スクロールを続ける動機が不足している。本プランでは2つの施策でこの課題を解消する。

## 変更対象ファイル

- [index.html](index.html) — HTML構造の変更
- [index.css](index.css) — CSSスタイルの追加

## 実装順序

**施策Bを先に実装 → 施策Aを後に実装** の順序で行う（行番号のズレを防ぐため、HTMLファイルの下側から上側に向かって挿入する）。

---

## 施策B: キャンペーンセクション新設（reasons直後）

### 挿入位置
```
217行目: </section>  ← 選ばれる3つの理由 終了
--- ★ ここに挿入 ---
219行目: <!-- 比較アンカリング -->
```

### HTML（挿入内容）
```html
<!-- キャンペーン告知（信頼構築後に提示） -->
<section class="campaign-moved section">
    <div class="container">
        <p class="campaign-lead">ここまで読んでいただいたあなたへ、特別なご案内です。</p>
        <div class="campaign-box">
            <h4 class="campaign-box__title">＼ 2月28日まで！先着5名限定 ／<br>新規入会キャンペーン</h4>
            <div class="campaign-box__item">
                <span class="campaign-box__label">入会金</span>
                <span class="campaign-box__price--old">11,000円</span>
                <span class="campaign-box__arrow">→</span>
                <span class="campaign-box__price--new">0円</span>
            </div>
            <div class="campaign-box__item">
                <span class="campaign-box__label">初月会費</span>
                <span class="campaign-box__price--old">9,980円</span>
                <span class="campaign-box__arrow">→</span>
                <span class="campaign-box__price--new">0円</span>
            </div>
            <div class="campaign-box__total">最大 20,980円 お得！</div>
        </div>
        <div class="u-text-center">
            <a href="https://piste-reserve.netlify.app/lp" class="btn btn--large" target="_blank">体調に合わせた無料体験を予約する</a>
        </div>
    </div>
</section>
```

### 設計方針
- 既存のオファーセクション（最下部）の `.campaign-box` スタイルを**そのまま再利用**
- CTAボタンも既存の `btn btn--large` クラスを使用（新規CSSクラス不要）
- 最下部のオファーセクションも**そのまま残す**（両方に存在する形）

### CSS追加内容
```css
/* Campaign Moved Section */
.campaign-moved {
    background-color: #fff9f0;
}

.campaign-lead {
    text-align: center;
    font-size: 14px;
    color: #666666;
    margin-bottom: 16px;
    padding: 0 20px;
}
```

---

## 施策A: 信頼バッジセクション新設（ヒーロー直下）

### 挿入位置
```
79行目: </section>  ← ヒーロー終了
--- ★ ここに挿入 ---
81行目: <!-- 共感・問題提起 -->
```

### HTML（挿入内容 — 2つの独立した兄弟要素）
```html
<!-- 信頼バッジセクション -->
<section class="trust-badges">
    <div class="trust-badge-item">
        <div class="trust-badge-icon">🏥</div>
        <div class="trust-badge-main">医療機関と連携</div>
        <div class="trust-badge-sub">整形外科医が監修</div>
    </div>
    <div class="trust-badge-item">
        <div class="trust-badge-icon">📊</div>
        <div class="trust-badge-main">継続率 92%</div>
        <div class="trust-badge-sub">無理なく続けられる</div>
    </div>
    <div class="trust-badge-item">
        <div class="trust-badge-icon">👥</div>
        <div class="trust-badge-main">40〜60代専門</div>
        <div class="trust-badge-sub">同世代だけの安心空間</div>
    </div>
</section>

<div class="scroll-hint">
    <span class="scroll-hint-arrow">↓</span> あなたに合ったプランを見る
</div>
```

**注意**: `<section class="trust-badges">` と `<div class="scroll-hint">` は**別々の兄弟要素**（入れ子ではない）。

### CSS追加内容
```css
/* Trust Badges Section */
.trust-badges {
    display: flex;
    justify-content: space-around;
    align-items: center;
    padding: 20px 16px;
    background: #FFFFFF;
    border-top: none;
}

.trust-badge-item {
    text-align: center;
    flex: 1;
    padding: 8px;
}

.trust-badge-icon {
    font-size: 28px;
    margin-bottom: 6px;
}

.trust-badge-main {
    font-size: 14px;
    font-weight: 700;
    color: #333333;
    margin-bottom: 2px;
}

.trust-badge-sub {
    font-size: 11px;
    color: #666666;
}

/* Scroll Hint Animation */
.scroll-hint {
    text-align: center;
    padding: 12px 0 8px;
    color: #FF6B35;
    font-size: 13px;
    font-weight: 600;
}

.scroll-hint-arrow {
    display: inline-block;
    animation: scrollBounce 1.5s ease infinite;
}

@keyframes scrollBounce {
    0%, 20%, 50%, 80%, 100% {
        transform: translateY(0);
    }
    40% {
        transform: translateY(-8px);
    }
    60% {
        transform: translateY(-4px);
    }
}
```

**注意**: アニメーション名は `scrollBounce`（既存の `@keyframes bounce`（CSS 820行目）との衝突を回避）。`.scroll-hint-arrow` の `animation` プロパティも `scrollBounce` を使用。

---

## 既存要素への影響

| 要素 | 影響 |
|:---|:---|
| ヒーロー内キャンペーンバッジ（赤丸） | 変更なし |
| キャンペーンモーダル（JS制御） | 変更なし（getElementById使用、DOM位置非依存） |
| オファーセクション（最下部） | 変更なし（キャンペーン内容が2箇所に存在する形） |
| フローティングCTA | 変更なし |

## CSS挿入位置

- 信頼バッジCSS → `index.css` のヒーローCSS（448行目付近）の直後に挿入
- キャンペーンセクションCSS → `index.css` の既存Offer/Campaign CSS（1237行目付近）の直後に挿入

## 検証方法

1. ブラウザでindex.htmlを開き、セクション順序が正しいことを確認
   - ヒーロー → 信頼バッジ → スクロールヒント → お悩み共感 → ...
   - ... → 選ばれる3つの理由 → キャンペーン告知 → 価格比較 → ...
2. モバイル表示（768px以下）で3バッジが横並びで表示されることを確認
3. スクロールヒントのバウンスアニメーションが動作することを確認
4. キャンペーンモーダル（ヒーロー内バッジクリック）が正常に動作することを確認
5. 新規キャンペーンセクションのCTAリンク先が `https://piste-reserve.netlify.app/lp` であることを確認
6. 既存の価格比較グラフのバウンスアニメーションに影響がないことを確認
