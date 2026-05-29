# GRKD-Tegaki 1段 手書きアニメーション用テンプレート

## 先にやること

`GRKD-Tegaki::1段` だけを変えたい場合、既存の `GRKD-Tegaki` ノートタイプをそのまま編集しないでください。同じノートタイプを使う `2段`、`3段`、`4段` にもテンプレート変更が広がります。

安全な手順:

1. Ankiで `ツール > ノートタイプを管理` を開く。
2. 既存の `GRKD-Tegaki` を選び、`複製` する。
3. 複製したノートタイプ名を `GRKD-Tegaki-1dan-Tegaki` にする。
4. 複製先のカードタイプが1枚だけであることを確認する。
5. 複製先に `ContextHint` フィールドを追加する。ヒント文は後で入れるので、今は空で大丈夫です。
6. 下のテンプレートを `GRKD-Tegaki-1dan-Tegaki` にだけ貼る。
7. Ankiのブラウザで `deck:GRKD-Tegaki::1段` を検索。
8. 表示された3278件を全選択。
9. `ノート > ノートタイプを変更`。
10. 変更先を `GRKD-Tegaki-1dan-Tegaki` にする。
11. 既存フィールドは同名のまま対応させ、`ContextHint` は空欄のままで進める。
12. カード数は1枚のままにする。
13. 変更後、まず1枚だけ開いてFront/Back表示とスケジュールが残っていることを確認する。

この手順ならカードを別デッキへ作り直さず、既存カードのスケジュールを保ったまま、1段だけ見た目を手書き練習用にできます。

## Front Template

```html
<div id="container">
  <div id="reading">{{Reading}}</div>
  <div id="audio">{{Audio}}</div>
  {{#ContextHint}}
    <div id="context-hint">
      <div class="hint-label">文脈</div>
      <div class="hint-text">{{ContextHint}}</div>
    </div>
  {{/ContextHint}}
</div>
```

## Back Template

```html
<div id="container">
  {{FrontSide}}
  <hr>
  <div id="word">{{Word}}</div>
  <div id="stroke-order-data" style="display:none">{{Word}}</div>
  <div id="stroke-order"></div>
  <div id="kvg-status"></div>
  <div id="kvg-credit">Stroke order: KanjiVG © Ulrich Apel, CC BY-SA 3.0</div>
</div>

<script>
(function () {
  const target = document.getElementById("stroke-order");
  const status = document.getElementById("kvg-status");
  const data = document.getElementById("stroke-order-data");
  if (!target) return;

  const word = data ? data.textContent || "" : "";
  // CJK Unified + Extension A + CJK Compatibility Ideographs.
  const kanji = Array.from(word).filter((ch) => /[\u3400-\u9fff\uf900-\ufaff]/u.test(ch));

  if (kanji.length === 0) {
    status.textContent = "漢字なし";
    return;
  }

  // KanjiVG file rule: U+XXXX -> five-digit lowercase hex + .svg
  function fileNameFor(ch) {
    return ch.codePointAt(0).toString(16).padStart(5, "0") + ".svg";
  }

  const strokeDelaySeconds = 0.48;
  const strokeDurationSeconds = 1.35;
  const kanjiGapSeconds = 0.45;

  function prepareSvg(svg, ch, startDelaySeconds) {
    svg.removeAttribute("width");
    svg.removeAttribute("height");
    svg.setAttribute("viewBox", svg.getAttribute("viewBox") || "0 0 109 109");
    svg.classList.add("kvg-svg");
    svg.setAttribute("aria-label", ch + " stroke order");

    const paths = Array.from(svg.querySelectorAll("path"));
    paths.forEach((path, strokeIndex) => {
      const length = Math.ceil(path.getTotalLength ? path.getTotalLength() : 180);
      path.classList.add("kvg-stroke");
      path.style.strokeDasharray = String(length);
      path.style.strokeDashoffset = String(length);
      path.style.animationDelay = `${startDelaySeconds + strokeIndex * strokeDelaySeconds}s`;
    });

    return svg;
  }

  async function loadKanji(ch) {
    const url = "https://raw.githubusercontent.com/KanjiVG/kanjivg/r20250816/kanji/" + fileNameFor(ch);
    const response = await fetch(url);
    if (!response.ok) throw new Error(ch + " not found");
    const text = await response.text();
    const doc = new DOMParser().parseFromString(text, "image/svg+xml");
    const svg = doc.querySelector("svg");
    if (!svg) throw new Error(ch + " invalid svg");

    return { ch, svg };
  }

  Promise.allSettled(kanji.map(loadKanji))
    .then((results) => {
      let nextKanjiDelaySeconds = 0;
      const nodes = results
        .filter((result) => result.status === "fulfilled")
        .map((result) => {
          const wrap = document.createElement("div");
          const paths = Array.from(result.value.svg.querySelectorAll("path"));
          wrap.className = "kvg-card";
          wrap.appendChild(prepareSvg(result.value.svg, result.value.ch, nextKanjiDelaySeconds));
          nextKanjiDelaySeconds += paths.length * strokeDelaySeconds + strokeDurationSeconds + kanjiGapSeconds;
          return wrap;
        });
      const failed = results.filter((result) => result.status === "rejected").length;
      target.replaceChildren(...nodes);
      status.textContent = failed > 0 ? failed + "字の書き順を読み込めませんでした" : "";
    });
})();
</script>
```

## Styling

```css
.card {
  margin: 0;
  background: #111;
  color: #f7f7f7;
  font-family: "UD Digi Kyokasho NK", "Yu Gothic", sans-serif;
  text-align: center;
}

#container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 22px;
  padding: 32px 16px;
}

#reading {
  font-size: clamp(44px, 9vw, 72px);
  line-height: 1.2;
}

#audio {
  font-size: 24px;
}

#context-hint {
  width: min(88vw, 620px);
  padding: 15px 18px 16px;
  border: 1px solid rgba(255, 255, 255, 0.16);
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.07);
  color: #f3f3f3;
  text-align: left;
  box-shadow: 0 14px 36px rgba(0, 0, 0, 0.18);
}

.hint-label {
  margin-bottom: 6px;
  color: #aaa;
  font-size: 12px;
  letter-spacing: 0.14em;
}

.hint-text {
  font-size: clamp(18px, 4vw, 28px);
  line-height: 1.55;
}

#word {
  font-size: clamp(56px, 11vw, 96px);
  line-height: 1.1;
}

#stroke-order {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 16px;
  width: min(94vw, 900px);
}

.kvg-card {
  width: 190px;
  height: 190px;
  padding: 12px;
  border-radius: 18px;
  background: #f7f7f7;
}

.kvg-svg {
  width: 100%;
  height: 100%;
}

.kvg-svg path {
  fill: none !important;
  stroke: #111 !important;
  stroke-width: 3 !important;
  stroke-linecap: round !important;
  stroke-linejoin: round !important;
}

.kvg-stroke {
  animation-delay: 0s;
  animation: draw-stroke 1.35s ease forwards;
}

@keyframes draw-stroke {
  to {
    stroke-dashoffset: 0;
  }
}

#kvg-status {
  min-height: 1.2em;
  color: #aaa;
  font-size: 14px;
}

#kvg-credit {
  color: #777;
  font-size: 11px;
}

hr {
  width: min(92vw, 760px);
  border: 0;
  border-top: 1px solid #555;
}
```

## 注意

- この版はKanjiVGのSVGをGitHubから読み込むので、表示にはネット接続が必要です。
- AnkiDroid / AnkiMobileではJavaScriptや外部fetchの挙動がPC版と違う可能性があります。まずPC版Ankiで1段の数枚を確認してください。
- `ContextHint` が空のカードでは、Frontに文脈ヒント欄は出ません。
- 点画が中抜きっぽく見える漢字があります。これはSVGの線をアニメーションさせるための見た目上のトレードオフです。
- 本番でオフライン安定化したい場合は、KanjiVG SVGを `collection.media` に入れて、ローカル参照版へ切り替えます。
