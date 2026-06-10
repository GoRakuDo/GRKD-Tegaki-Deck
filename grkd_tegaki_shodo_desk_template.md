# GRKD-Tegaki 静かな書道机テンプレート案

## コンセプト

読む、思い出す、実際に書く、裏で確認する。  
画面は主役ではなく、机の上に置いた紙くらい静かにします。

- Light: 生成りの和紙、薄い罫線、濃い墨
- Dark: 硯のような墨色、白い紙カード、控えめな朱色
- Logo: 主役にせず、小さい判子/透かしとして使う

## 先にやること

ロゴを使う場合は、Ankiのメディアフォルダに次の2つを入れるのがおすすめです。

```text
logo_black.svg
logo_white.svg
```

元ファイル例:

```text
E:\Libraries\Documents\Logo_BW.svg
```

Lightでは黒ロゴ、Darkでは白ロゴを表示します。ロゴなしでもカードは動きます。ロゴを入れない場合は、テンプレート内の `grkd-seal` の `<span>` を削除してください。

## Front Template

```html
<main class="grkd-card grkd-front-card">
  <header class="grkd-brand">
    <a class="grkd-logo-link" href="https://gorakudo.org" target="_blank" rel="noopener" aria-label="GoRakuDo公式サイトを開く">
      <span class="grkd-seal grkd-seal-light" aria-hidden="true"></span>
      <span class="grkd-seal grkd-seal-dark" aria-hidden="true"></span>
    </a>
  </header>

  <section class="grkd-prompt" aria-label="問題">
    <div class="grkd-reading" lang="ja">{{Reading}}</div>
    <div class="grkd-audio">{{Audio}}</div>
  </section>

  {{#ContextHint}}
    <section class="grkd-hint" aria-label="手がかり">
      <div class="grkd-hint-text" lang="ja">{{ContextHint}}</div>
    </section>
  {{/ContextHint}}
</main>
```

## Back Template

```html
<main class="grkd-card grkd-back-card">
  <section class="grkd-front-echo">
    {{FrontSide}}
  </section>

  <section class="grkd-answer" aria-label="答え">
    <div class="grkd-word" lang="ja">{{Word}}</div>
  </section>

  <section class="grkd-stroke-panel" aria-label="筆順">
    <div id="stroke-order-data" style="display:none">{{Word}}</div>
    <div id="stroke-order" class="grkd-stroke-grid"></div>
    <div id="kvg-status" class="grkd-status"></div>
  </section>

  <footer class="grkd-footer">
    <span>筆順データ: KanjiVG © Ulrich Apel, CC BY-SA 3.0</span>
    <a class="grkd-logo-link" href="https://gorakudo.org" target="_blank" rel="noopener" aria-label="GoRakuDo公式サイトを開く">
      <span class="grkd-footer-seal grkd-seal-light" aria-hidden="true"></span>
      <span class="grkd-footer-seal grkd-seal-dark" aria-hidden="true"></span>
    </a>
  </footer>
</main>

<script>
(function () {
  const target = document.getElementById("stroke-order");
  const status = document.getElementById("kvg-status");
  const data = document.getElementById("stroke-order-data");
  if (!target) return;

  const word = data ? data.textContent || "" : "";
  const kanji = Array.from(word).filter(function (ch) {
    return /[\u3400-\u9fff\uf900-\ufaff]/u.test(ch);
  });

  if (kanji.length === 0) {
    if (status) status.textContent = "漢字なし";
    return;
  }

  if (status) status.textContent = "筆順を読み込み中…";

  const skeletons = kanji.map(function () {
    const skeleton = document.createElement("div");
    const mark = document.createElement("div");
    skeleton.className = "kvg-card kvg-skeleton";
    skeleton.setAttribute("aria-hidden", "true");
    mark.className = "kvg-skeleton-mark";
    skeleton.appendChild(mark);
    return skeleton;
  });
  target.replaceChildren.apply(target, skeletons);

  function fileNameFor(ch) {
    return ch.codePointAt(0).toString(16).padStart(5, "0") + ".svg";
  }

  const strokeDelaySeconds = 0.48;
  const strokeDurationSeconds = 1.35;
  const kanjiGapSeconds = 0.52;
  const cache = window.__grkdKanjiSvgCache || (window.__grkdKanjiSvgCache = Object.create(null));
  let replayData = [];

  function prepareSvg(svg, ch, startDelaySeconds) {
    svg.removeAttribute("width");
    svg.removeAttribute("height");
    svg.setAttribute("viewBox", svg.getAttribute("viewBox") || "0 0 109 109");
    svg.classList.add("kvg-svg");
    svg.setAttribute("aria-label", ch + " 筆順");

    const paths = Array.from(svg.querySelectorAll("path"));
    paths.forEach(function (path, strokeIndex) {
      const length = Math.ceil(path.getTotalLength ? path.getTotalLength() : 180);
      path.classList.add("kvg-stroke");
      path.style.strokeDasharray = String(length);
      path.style.strokeDashoffset = String(length);
      path.style.animationDelay = String(startDelaySeconds + strokeIndex * strokeDelaySeconds) + "s";
    });

    return svg;
  }

  async function loadKanji(ch) {
    if (cache[ch]) {
      const cachedDoc = new DOMParser().parseFromString(cache[ch], "image/svg+xml");
      const cachedSvg = cachedDoc.querySelector("svg");
      if (!cachedSvg) throw new Error(ch + " invalid cached svg");
      return { ch: ch, svg: cachedSvg, cached: true };
    }

    const url = "https://raw.githubusercontent.com/KanjiVG/kanjivg/r20250816/kanji/" + fileNameFor(ch);
    const response = await fetch(url);
    if (!response.ok) throw new Error(ch + " not found");
    const text = await response.text();
    cache[ch] = text;
    const doc = new DOMParser().parseFromString(text, "image/svg+xml");
    const svg = doc.querySelector("svg");
    if (!svg) throw new Error(ch + " invalid svg");
    return { ch: ch, svg: svg, cached: false };
  }

  function buildStrokeNodes(items) {
    let nextKanjiDelaySeconds = 0;
    return items.map(function (item) {
      const wrap = document.createElement("div");
      const doc = new DOMParser().parseFromString(item.svgText, "image/svg+xml");
      const svg = doc.querySelector("svg");
      if (!svg) throw new Error(item.ch + " invalid replay svg");
      const paths = Array.from(svg.querySelectorAll("path"));
      wrap.className = "kvg-card";
      wrap.appendChild(prepareSvg(svg, item.ch, nextKanjiDelaySeconds));
      nextKanjiDelaySeconds += paths.length * strokeDelaySeconds + strokeDurationSeconds + kanjiGapSeconds;
      return wrap;
    });
  }

  function replayStrokeOrder() {
    if (replayData.length === 0) return;
    const nodes = buildStrokeNodes(replayData);
    target.replaceChildren.apply(target, nodes);
  }

  target.addEventListener("click", replayStrokeOrder);

  Promise.allSettled(kanji.map(loadKanji)).then(function (results) {
    replayData = results
      .filter(function (result) { return result.status === "fulfilled"; })
      .map(function (result) {
        return { ch: result.value.ch, svgText: cache[result.value.ch] };
      });

    const nodes = buildStrokeNodes(replayData);

    const failed = results.filter(function (result) { return result.status === "rejected"; }).length;
    const loaded = nodes.length;
    const fromCache = results.filter(function (result) {
      return result.status === "fulfilled" && result.value.cached;
    }).length;
    target.replaceChildren.apply(target, nodes);
    if (status) {
      if (failed > 0 && loaded === 0) {
        status.textContent = "筆順がありません";
      } else if (failed > 0) {
        status.textContent = "一部の筆順がありません";
      } else if (fromCache > 0) {
        status.textContent = "";
      } else {
        status.textContent = "";
      }
    }
  });
})();
</script>
```

## Styling

```css
@import url("https://cdn.jsdelivr.net/npm/gen-interface-jp@latest/cdn/400.css");
@import url("https://cdn.jsdelivr.net/npm/gen-interface-jp@latest/cdn/700.css");

html,
body,
#qa,
.card {
  --grkd-paper: oklch(0.965 0.018 82);
  --grkd-paper-soft: oklch(0.99 0.009 82);
  --grkd-ink: oklch(0.22 0.012 72);
  --grkd-muted: oklch(0.50 0.018 75);
  --grkd-faint: oklch(0.74 0.022 78);
  --grkd-line: oklch(0.74 0.024 78 / 0.46);
  --grkd-panel: oklch(0.985 0.011 82 / 0.82);
  --grkd-accent: oklch(0.45 0.18 265);
  --grkd-accent-soft: oklch(0.60 0.14 265 / 0.10);
  --grkd-clear: oklch(0.60 0.14 265 / 0);
  --grkd-shadow: oklch(0.27 0.018 72 / 0.16);
  --grkd-stroke-paper: oklch(0.99 0.009 82);
  --grkd-stroke-ink: oklch(0.18 0.012 72);
  --grkd-stroke-grid: oklch(0.45 0.18 265 / 0.10);
  --grkd-stroke-border: oklch(0.74 0.024 78 / 0.42);

  margin: 0;
  min-height: 100%;
  background:
    radial-gradient(circle at 50% 0%, var(--grkd-accent-soft), var(--grkd-clear) 28rem),
    linear-gradient(180deg, var(--grkd-paper-soft), var(--grkd-paper));
  color: var(--grkd-ink);
  font-family: "Gen Interface JP", "Yu Gothic", "Hiragino Kaku Gothic ProN", sans-serif;
  text-align: center;
}

html,
body,
#qa {
  min-height: 100vh;
  margin: 0;
  scrollbar-gutter: stable;
}

body {
  overflow-y: scroll;
}

.card.night_mode,
.nightMode .card,
.night_mode .card,
html.night-mode,
html.night_mode,
body.night_mode,
body.nightMode,
html.night-mode #qa,
html.night_mode #qa,
body.night_mode #qa,
body.nightMode #qa {
  --grkd-paper: oklch(0.18 0.012 72);
  --grkd-paper-soft: oklch(0.22 0.014 72);
  --grkd-ink: oklch(0.93 0.018 82);
  --grkd-muted: oklch(0.70 0.018 78);
  --grkd-faint: oklch(0.43 0.018 78);
  --grkd-line: oklch(0.93 0.018 82 / 0.16);
  --grkd-panel: oklch(0.98 0.006 82 / 0.055);
  --grkd-accent: oklch(0.68 0.16 265);
  --grkd-accent-soft: oklch(0.68 0.16 265 / 0.14);
  --grkd-clear: oklch(0.68 0.16 265 / 0);
  --grkd-shadow: oklch(0.08 0.01 72 / 0.40);
  --grkd-stroke-paper: oklch(0.96 0.018 82);
  --grkd-stroke-ink: oklch(0.18 0.012 72);
  --grkd-stroke-grid: oklch(0.68 0.16 265 / 0.12);
  --grkd-stroke-border: oklch(0.93 0.018 82 / 0.18);
  background:
    radial-gradient(circle at 50% 0%, var(--grkd-accent-soft), var(--grkd-clear) 30rem),
    linear-gradient(180deg, var(--grkd-paper-soft), var(--grkd-paper));
}

.card.night_mode,
.nightMode .card,
.night_mode .card {
  --grkd-paper: oklch(0.18 0.012 72);
  --grkd-paper-soft: oklch(0.22 0.014 72);
  --grkd-ink: oklch(0.93 0.018 82);
  --grkd-muted: oklch(0.70 0.018 78);
  --grkd-faint: oklch(0.43 0.018 78);
  --grkd-line: oklch(0.93 0.018 82 / 0.16);
  --grkd-panel: oklch(0.98 0.006 82 / 0.055);
  --grkd-accent: oklch(0.68 0.16 265);
  --grkd-accent-soft: oklch(0.68 0.16 265 / 0.14);
  --grkd-clear: oklch(0.68 0.16 265 / 0);
  --grkd-shadow: oklch(0.08 0.01 72 / 0.40);
  --grkd-stroke-paper: oklch(0.96 0.018 82);
  --grkd-stroke-ink: oklch(0.18 0.012 72);
  --grkd-stroke-grid: oklch(0.68 0.16 265 / 0.12);
  --grkd-stroke-border: oklch(0.93 0.018 82 / 0.18);
  background:
    radial-gradient(circle at 50% 0%, var(--grkd-accent-soft), var(--grkd-clear) 30rem),
    linear-gradient(180deg, var(--grkd-paper-soft), var(--grkd-paper));
}

.grkd-card {
  box-sizing: border-box;
  width: min(94vw, 760px);
  min-height: auto;
  margin: 0 auto;
  padding: clamp(22px, 5vw, 46px) 18px 34px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.grkd-brand {
  min-height: 34px;
  margin-bottom: clamp(18px, 5vw, 42px);
}

.grkd-logo-link {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  color: inherit;
  text-decoration: none;
}

.grkd-seal,
.grkd-footer-seal {
  display: inline-block;
  width: 30px;
  height: 30px;
  opacity: 0.28;
  background-position: center;
  background-repeat: no-repeat;
  background-size: contain;
}

.grkd-seal-light {
  background-image: url("https://raw.githubusercontent.com/GoRakuDo/GRKD-Tegaki-Deck/main/logo_black.svg");
}

.grkd-seal-dark {
  display: none;
  background-image: url("https://raw.githubusercontent.com/GoRakuDo/GRKD-Tegaki-Deck/main/logo_black.svg");
  filter: invert(1);
}

.card.night_mode .grkd-seal-light,
.nightMode .card .grkd-seal-light,
.night_mode .card .grkd-seal-light,
body.nightMode .grkd-seal-light,
body.night_mode .grkd-seal-light,
html.night-mode .grkd-seal-light,
html.night_mode .grkd-seal-light {
  display: none;
}

.card.night_mode .grkd-seal-dark,
.nightMode .card .grkd-seal-dark,
.night_mode .card .grkd-seal-dark,
body.nightMode .grkd-seal-dark,
body.night_mode .grkd-seal-dark,
html.night-mode .grkd-seal-dark,
html.night_mode .grkd-seal-dark {
  display: inline-block;
}

@media (prefers-color-scheme: dark) {
  .grkd-seal-light {
    display: none;
  }

  .grkd-seal-dark {
    display: inline-block;
  }
}

.grkd-prompt {
  width: 100%;
}

.grkd-reading {
  font-size: clamp(42px, 10vw, 78px);
  line-height: 1.12;
  letter-spacing: 0.03em;
  color: var(--grkd-ink);
  font-family: "UD Digi Kyokasho NK", "Yu Mincho", "Hiragino Mincho ProN", "Noto Serif CJK JP", serif;
  text-wrap: balance;
}

.grkd-audio {
  margin-top: 20px;
  min-height: 36px;
  opacity: 0.86;
}

.grkd-hint {
  box-sizing: border-box;
  width: min(92vw, 640px);
  margin-top: clamp(24px, 7vw, 44px);
  padding: 17px 20px 19px;
  border: 1px solid var(--grkd-line);
  border-radius: 20px;
  background: var(--grkd-panel);
  box-shadow: 0 18px 42px var(--grkd-shadow);
  text-align: left;
}

.grkd-hint-text {
  color: var(--grkd-ink);
  font-family: "Gen Interface JP", "Yu Gothic", "Hiragino Kaku Gothic ProN", sans-serif;
  font-size: clamp(18px, 4.2vw, 27px);
  line-height: 1.58;
  letter-spacing: 0.02em;
}

.grkd-front-echo {
  width: 100%;
  padding-bottom: clamp(24px, 6vw, 42px);
  margin-bottom: clamp(24px, 6vw, 42px);
  border-bottom: 1px solid var(--grkd-line);
}

.grkd-front-echo .grkd-card {
  width: 100%;
  margin: 0;
  padding: 0;
}

.grkd-answer {
  width: 100%;
}

.grkd-word {
  margin-top: 12px;
  font-size: clamp(58px, 13vw, 112px);
  line-height: 1.04;
  letter-spacing: 0.04em;
  color: var(--grkd-ink);
  font-family: "UD Digi Kyokasho NK", "Yu Mincho", "Hiragino Mincho ProN", "Noto Serif CJK JP", serif;
  text-wrap: balance;
}

.grkd-stroke-panel {
  width: 100%;
  margin-top: clamp(26px, 7vw, 48px);
}

.grkd-stroke-grid {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 16px;
  width: 100%;
  cursor: pointer;
}

.kvg-card {
  box-sizing: border-box;
  width: clamp(142px, 34vw, 204px);
  height: clamp(142px, 34vw, 204px);
  padding: 13px;
  border: 1px solid var(--grkd-stroke-border);
  border-radius: 20px;
  background:
    linear-gradient(var(--grkd-stroke-grid) 1px, var(--grkd-clear) 1px),
    linear-gradient(90deg, var(--grkd-stroke-grid) 1px, var(--grkd-clear) 1px),
    var(--grkd-stroke-paper);
  background-size: 50% 50%;
  box-shadow: 0 14px 36px var(--grkd-shadow);
}

.kvg-skeleton {
  position: relative;
  overflow: hidden;
}

.kvg-skeleton::after {
  content: "";
  position: absolute;
  inset: 0;
  transform: translateX(-100%);
  background: linear-gradient(
    90deg,
    var(--grkd-clear),
    oklch(1 0 0 / 0.42),
    var(--grkd-clear)
  );
  animation: grkd-skeleton-sweep 1.45s ease-in-out infinite;
}

.kvg-skeleton-mark {
  width: 100%;
  height: 100%;
  border-radius: 13px;
  background:
    linear-gradient(135deg, var(--grkd-clear) 46%, var(--grkd-stroke-grid) 47%, var(--grkd-stroke-grid) 53%, var(--grkd-clear) 54%),
    linear-gradient(45deg, var(--grkd-clear) 46%, var(--grkd-stroke-grid) 47%, var(--grkd-stroke-grid) 53%, var(--grkd-clear) 54%),
    radial-gradient(circle at 50% 50%, var(--grkd-stroke-grid) 0 10%, var(--grkd-clear) 11%);
}

.kvg-svg {
  width: 100%;
  height: 100%;
  display: block;
}

.kvg-svg path {
  fill: none !important;
  stroke: var(--grkd-stroke-ink) !important;
  stroke-width: 3 !important;
  stroke-linecap: round !important;
  stroke-linejoin: round !important;
}

.kvg-stroke {
  animation: grkd-draw-stroke 1.35s cubic-bezier(0.32, 0.72, 0, 1) forwards;
}

@keyframes grkd-draw-stroke {
  to {
    stroke-dashoffset: 0;
  }
}

@keyframes grkd-skeleton-sweep {
  to {
    transform: translateX(100%);
  }
}

.grkd-status {
  min-height: 1.3em;
  margin-top: 14px;
  color: var(--grkd-muted);
  font-family: "Gen Interface JP", "Yu Gothic", "Hiragino Kaku Gothic ProN", sans-serif;
  font-size: 13px;
}

.grkd-footer {
  box-sizing: border-box;
  width: min(92vw, 640px);
  margin-top: auto;
  padding-top: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  color: var(--grkd-muted);
  font-family: "Gen Interface JP", "Yu Gothic", "Hiragino Kaku Gothic ProN", sans-serif;
  font-size: 11px;
  line-height: 1.5;
}

.grkd-footer-seal {
  width: 22px;
  height: 22px;
}

hr {
  display: none;
}

@media screen and (max-width: 560px) {
  .grkd-stroke-grid {
    gap: 12px;
  }

  .kvg-card {
    width: min(43vw, 172px);
    height: min(43vw, 172px);
  }
}

@media screen and (max-width: 480px) {
  .grkd-card {
    width: 100%;
    min-height: auto;
    padding: 22px 14px 28px;
  }

  .grkd-hint {
    width: 100%;
    border-radius: 17px;
  }

  .kvg-card {
    border-radius: 16px;
  }

  .grkd-footer {
    width: 100%;
    flex-direction: column;
    gap: 8px;
  }
}
```

## 注意

- この版もKanjiVGのSVGをGitHubから読み込むため、表示にはネット接続が必要です。
- 読み込み中は筆順マスのSkeletonを表示します。同じAnkiセッション内では、一度読んだ漢字SVGをメモリ上で再利用します。筆順マスをクリック/タップするとアニメーションを最初から再生します。
- AnkiDroid / AnkiMobileまで安定させるなら、将来的にKanjiVG SVGをAnkiメディアへ入れるローカル版に切り替えるのがおすすめです。
- `ContextHint` が空のカードでは、Frontにお題カードは出ません。
- ロゴは公開済みGitHubのSVGを読み込みます。Lightでは黒ロゴ、Darkでは黒SVGをCSS反転して白ロゴとして表示します。AnkiのNight Mode classに加えて、OS/ブラウザの `prefers-color-scheme: dark` でも白ロゴへ切り替えます。ロゴをクリック/タップすると `https://gorakudo.org` を開きます。オフラインでも表示したい場合は、`logo_black.svg` をAnkiメディアへ入れて、CSSのURLをローカル名へ戻してください。
- Github公開時はKanjiVGのライセンス表記をREADMEにも入れてください。
