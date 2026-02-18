# フェーズ2 修正実装プラン

## Context
Clarityヒートマップ解析に基づくLP改善のフェーズ2（中期改善）。フェーズ1（特典バッジモーダル・ヒーローCTA強化・FAQ上部移動）は実装済み。フェーズ2の3項目（画像ギャラリー・スクロール非表示・価格上部移動）を実装する。

## 対象ファイル
- `index.html` — HTML構造の変更
- `index.css` — スタイル追加

---

## 修正案2-1: 画像Lightbox（自前軽量実装）

### 方針
既存のキャンペーンモーダル（`#campaignModal`）のパターンを流用し、画像拡大用のLightboxモーダルを1つ追加。各画像にクリックイベントでLightboxを開く。

### 対象画像（8枚）
| 画像 | 場所（行） | キャプション |
|---|---|---|
| hero_v2.png | L63 | 笑顔でカウンセリングを受ける様子 |
| intro_v2.png | L120 | 肩の不調を感じる様子 |
| solution_train_notebook.png | L186 | データに基づいた安全なトレーニング |
| stretch_v2.png | L207 | リラックスしたパーソナルストレッチ |
| reason_doctor.png | L227 | 医師と笑顔で握手するトレーナー |
| reason_science.png | L235 | タブレットでデータを説明するトレーナー |
| reason_open.png | L243 | 笑顔でジムに入店する会員様 |
| trainer_real.jpg | L430 | トレーナー石川卓 |

### HTML変更

**1. 各img要素にdata属性を追加（ラッパーdivは使わない）**

既存レイアウトを壊さないため、`<img>`要素自体に`data-lightbox-caption`属性を追加し、CSSでcursorを変更する方式にする。新しいdivでラップしない。

```html
<!-- 例: hero画像 -->
<img src="hero_v2.png" alt="笑顔でカウンセリングを受ける様子"
     data-lightbox-caption="笑顔でカウンセリングを受ける様子（タップで拡大）"
     class="lightbox-target">
```

特別な注意点:
- `hero_v2.png`: 親要素`.hero__image`内にキャンペーンバッジが絶対配置されているため、imgのみにクラス追加。ラップ不可
- `reason_*.png`: 親要素`.reason-card`に`overflow: hidden`があるため、imgのみにクラス追加。ラップ不可

**2. 各画像の下にヒントテキスト追加**

```html
<p class="lightbox-hint">
  <span class="lightbox-hint--touch">タップで拡大</span>
  <span class="lightbox-hint--mouse">クリックで拡大</span>
</p>
```

CSSでデバイスに応じて出し分け:
- タッチデバイス: 「タップで拡大」表示
- マウスデバイス: 「クリックで拡大」表示

**3. ページ末尾にLightboxモーダル追加（既存モーダルの後）**

```html
<div id="lightboxModal" class="lightbox" aria-hidden="true">
  <div class="lightbox__overlay" id="lightboxOverlay"></div>
  <div class="lightbox__content">
    <button class="lightbox__close" id="lightboxClose" aria-label="閉じる">&times;</button>
    <img class="lightbox__image" id="lightboxImage" src="" alt="">
    <p class="lightbox__caption" id="lightboxCaption"></p>
  </div>
</div>
```

### CSS追加
```css
/* Lightbox Target */
img.lightbox-target {
  cursor: pointer;
}

/* Lightbox Hint */
.lightbox-hint {
  text-align: center;
  font-size: 0.8rem;
  color: var(--text-light);
  margin-top: 8px;
}

/* デバイスによる出し分け */
.lightbox-hint--mouse { display: none; }
@media (hover: hover) and (pointer: fine) {
  .lightbox-hint--touch { display: none; }
  .lightbox-hint--mouse { display: inline; }
}

/* Lightbox Modal */
.lightbox {
  position: fixed;
  top: 0; left: 0;
  width: 100%; height: 100%;
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease, visibility 0.3s ease;
}

.lightbox--active {
  opacity: 1;
  visibility: visible;
}

.lightbox__overlay {
  position: absolute;
  top: 0; left: 0;
  width: 100%; height: 100%;
  background: rgba(0, 0, 0, 0.85);
  cursor: pointer;
}

.lightbox__content {
  position: relative;
  z-index: 10000;
  text-align: center;
  max-width: 95vw;
  max-height: 90vh;
}

.lightbox__image {
  max-width: 95vw;
  max-height: 80vh;
  object-fit: contain;
  border-radius: 8px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
}

.lightbox__caption {
  color: #fff;
  font-size: 0.95rem;
  margin-top: 15px;
}

.lightbox__close {
  position: absolute;
  top: -40px; right: 0;
  background: transparent;
  border: none;
  font-size: 2.5rem;
  color: #fff;
  cursor: pointer;
  width: 40px; height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
}
```

### JS追加
```js
// Lightbox制御
(function() {
  var lbModal = document.getElementById('lightboxModal');
  var lbImage = document.getElementById('lightboxImage');
  var lbCaption = document.getElementById('lightboxCaption');
  var lbClose = document.getElementById('lightboxClose');
  var lbOverlay = document.getElementById('lightboxOverlay');

  function openLightbox(src, alt, caption) {
    lbImage.src = src;
    lbImage.alt = alt;
    lbCaption.textContent = caption;
    lbModal.setAttribute('aria-hidden', 'false');
    lbModal.classList.add('lightbox--active');
    document.body.style.overflow = 'hidden'; // スクロールロック
    lbClose.focus(); // フォーカス移動
  }

  function closeLightbox() {
    lbModal.setAttribute('aria-hidden', 'true');
    lbModal.classList.remove('lightbox--active');
    document.body.style.overflow = '';
  }

  // 各画像のクリックイベント
  document.querySelectorAll('img.lightbox-target').forEach(function(img) {
    img.addEventListener('click', function() {
      openLightbox(img.src, img.alt, img.dataset.lightboxCaption || img.alt);
    });
  });

  // 閉じる: ×ボタン、オーバーレイクリック、Escapeキー
  lbClose.addEventListener('click', closeLightbox);
  lbOverlay.addEventListener('click', closeLightbox);
  document.addEventListener('keydown', function(e) {
    if (e.key === 'Escape' && lbModal.classList.contains('lightbox--active')) {
      closeLightbox();
    }
  });
})();
```

---

## 修正案2-2: スクロール開始後の非表示処理

### 方針
既存の `.scroll-hint` と `.voice-scroll-hint` にスクロール検知を追加。100px以上スクロールしたらフェードアウト＋pointer-events無効化。

### CSS追加
```css
.scroll-hint,
.voice-scroll-hint {
  transition: opacity 0.5s ease;
}
```

### JS追加（既存scriptタグ内）
```js
// スクロールヒント非表示
var scrollHintEl = document.querySelector('.scroll-hint');
var voiceScrollHintEl = document.querySelector('.voice-scroll-hint');
window.addEventListener('scroll', function() {
  var hidden = window.scrollY > 100;
  if (scrollHintEl) {
    scrollHintEl.style.opacity = hidden ? '0' : '1';
    scrollHintEl.style.pointerEvents = hidden ? 'none' : '';
  }
  if (voiceScrollHintEl) {
    voiceScrollHintEl.style.opacity = hidden ? '0' : '1';
    voiceScrollHintEl.style.pointerEvents = hidden ? 'none' : '';
  }
}, { passive: true });
```

---

## 修正案2-3: 簡易価格セクション追加

### 配置
`faq-early`（早期FAQ）の直後、`solution`（解決策）の直前に新セクション挿入。
目的: 価格の透明性を早期に見せてスクロール継続を促す。ページ下部の詳細料金プランへの導線。

### HTML追加（index.html L176の後、L178の前）
```html
<!-- 簡易価格ティーザー -->
<section class="price-teaser section">
  <div class="container u-text-center">
    <p class="price-teaser__label">大手パーソナルジムの <strong>1/10</strong> の価格</p>
    <p class="price-teaser__price">月額 <span class="price-teaser__amount">7,500</span><span class="price-teaser__unit">円〜</span></p>
    <p class="price-teaser__features">通い放題 ・ 医療提携 ・ 食事指導付き</p>
    <a href="#price" class="btn btn--price-teaser">プラン詳細を見る ↓</a>
  </div>
</section>
```

### CSS追加
```css
.price-teaser {
  background: linear-gradient(135deg, #fffbf0 0%, #fff 100%);
  padding: 40px 0;
}

.price-teaser__label {
  font-size: 1rem;
  color: var(--text-light);
  margin-bottom: 10px;
}

.price-teaser__label strong {
  color: var(--secondary-color);
  font-size: 1.2rem;
}

.price-teaser__price {
  font-size: 1.2rem;
  color: var(--primary-color);
  margin-bottom: 10px;
}

.price-teaser__amount {
  font-size: 2.8rem;
  font-weight: 800;
  color: var(--secondary-color);
}

.price-teaser__unit {
  font-size: 1.1rem;
  font-weight: 700;
}

.price-teaser__features {
  font-size: 0.95rem;
  color: var(--text-main);
  margin-bottom: 20px;
}

.btn--price-teaser {
  background: transparent;
  color: var(--primary-color);
  border: 2px solid var(--primary-color);
  padding: 12px 30px;
  font-size: 1rem;
}

.btn--price-teaser:hover {
  background: var(--primary-color);
  color: var(--white);
  box-shadow: 0 4px 12px rgba(26, 54, 93, 0.3);
}
```

---

## 実装順序
1. 修正案2-2（スクロール非表示）— 最小変更、即完了
2. 修正案2-3（簡易価格セクション）— HTML/CSS追加
3. 修正案2-1（画像Lightbox）— 最も変更量が多い

## 検証方法
ブラウザでindex.htmlを開き、以下を確認:
1. スクロールすると `scroll-hint` と `voice-scroll-hint` がフェードアウトし、トップに戻ると復帰すること
2. 早期FAQの後に簡易価格セクションが表示され、「プラン詳細を見る」で#priceにスムーズスクロールすること
3. 各画像をクリック/タップするとLightboxが開き、拡大画像とキャプションが表示されること
4. Lightboxの閉じ方が3通り機能すること（×ボタン、オーバーレイクリック、Escapeキー）
5. Lightbox表示中にページがスクロールしないこと（body overflow hidden）
6. モバイルでは「タップで拡大」、PCでは「クリックで拡大」と表示されること
7. モバイル（768px以下）でレイアウトが崩れないこと
